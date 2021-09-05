## Android下休眠唤醒的实现

平台：MT6757（P25）

版本：Android7.1.1

​	Android下的休眠唤醒，这次分为两部分实现：Android系统和Recovery模式。

### Android系统下的休眠唤醒

​	android系统下休眠唤醒都比较顺畅，基本用命令行可以验证得出，使用C语言写代码也可以实现。

- 休眠

  休眠可以使用shell命令进入：`echo mem > /sys/power/state`

  休眠机制的整体流程暂未研究，主要是suspend，具体代码在`kernel-4.4/kernel/power/suspend.c`

- 唤醒

  唤醒部分使用RTC Alarm的wakealarm，以下介绍下RTC。

  RTC子系统：

  ​	rtc子系统在目录`/sys/class/rtc/`下，会根据设备创建对应的目录。

  ​	rtc目录中的**wakealarm**文件内容是下次触发唤醒时间的时间，默认这个文件是没有值的，文件

  ​	的内容是需要设置的时间（秒数）

  ​	比如设置60s后唤醒：

  ​	`echo +60 > /sys/class/rtc/rtc0/wakealarm`

  ​	查看设置的时间：

  ​	`cat /sys/class/rtc/rtc0/wakealarm`

  PROCFS接口：

  ​	可以查看rtc的详细信息，`/proc/driver/rtc`这个文件记录了rtc的详细信息。

  比如在设置wakealarm之前，发送命令`cat /proc/driver/rtc`，得到以下信息

  ~~~
  rtc_time        : 00:00:33
  rtc_date        : 2010-01-01
  alrm_time       : 00:00:30
  alrm_date       : 1970-01-01
  alarm_IRQ       : no
  alrm_pending    : no
  update IRQ enabled      : no
  periodic IRQ enabled    : no
  periodic IRQ frequency  : 1
  max user IRQ frequency  : 64
  24hr            : yes
  ~~~

  设置wakealarm：`echo +60 >/sys/class/rtc/rtc0/wakealarm`

  再查看rtc信息：`cat /proc/driver/rtc`

  ~~~
  rtc_time        : 00:00:33
  rtc_date        : 2010-01-01
  alrm_time       : 00:01:33
  alrm_date       : 2010-01-01
  alarm_IRQ       : yes
  alrm_pending    : no
  update IRQ enabled      : no
  periodic IRQ enabled    : no
  periodic IRQ frequency  : 1
  max user IRQ frequency  : 64
  24hr            : yes
  ~~~

  >对比上面两次获取rtc信息可以看出区别：在设置wakealarm之后，rtc信息中的**alarm_IRQ**变成了yes，**rtc_time**表示当前的rtc时间，**alrm_time**表示唤醒的时间，对比可知刚好是一分钟，也就是设置的wakealarm的+60秒。

- 总结

  通过以上的说明，可以得出结论，在Android系统上操作休眠唤醒，只需要以下几步：

  1. 设置wakealarm时间，设置唤醒时间。
  2. 向/sys/power/state写入mem，系统进入休眠

### Recovery模式下的休眠唤醒

​	正常来说，Recovery模式下休眠唤醒应该是和Android系统下的一样的操作方式的。（实质上也差不多），但是我在用recovery模式下进行，**在向/sys/power/state下写入mem时，总是报错：Operation not permitted**，但是open和read这个文件却是OK的，操作rtc也是可以的。

#### 1. 分析流程

​	后来仔细查看log，研究这个休眠的流程，有这么一句log：**SPM FIRMWARE IS NOT READY**

​	SPM = System Power Manager

​	因为整个系统不只是有MCU，还包括其他子系统，当进入休眠后，整个系统就靠一颗SCP：SPM来控制休眠唤醒的流程，通白点讲就是：其实有两个处理器，进入休眠后就由另一个低功耗点的处理器维持一些按键，中断，以用来唤醒主处理器。

​	而从log看到SPM Firmware是没有ready的，去找到打印这句话的文件，在`kernel-4.4/drivers/misc/mediatek/base/power/spm_v2/mtk_sleep.c`函数slp_suspend_ops_enter中。

~~~c
...

#if !defined(CONFIG_FPGA_EARLY_PORTING)
	if (!spm_load_firmware_status()) {
		slp_error("SPM FIRMWARE IS NOT READY\n");
		ret = -EPERM;
		goto LEAVE_SLEEP;
	}
#endif

...
~~~

​	发现在这里给return了，退出sleep，尝试直接注释这一段，发现去write mem到/sys/power/state不再报不能操作了，能正常写了，但是系统却挂了，这里猜测应该是spm在recovery模式下没有load相应的固件，所以进入休眠后根本没法去运行，所以只能继续分析下去。

​	继续追踪，查看函数spm_load_firmware_status(),函数在`mtk_spm.c`

~~~c
int spm_load_firmware_status(void)
{
	return dyna_load_pcm_done;
}
~~~

​	函数中只是返回一个状态`dyna_load_pcm_done`，在**mtk_spm.c**中全局搜索这个变量，发现真正给这个变量赋值为1的，只有一个函数：**spm_load_pcm_firmware()**,说明应该是需要先调用这个函数，继续往上追寻。

​	函数**spm_load_pcm_firmware_nodev()**调用了**spm_load_pcm_firmware()**

~~~C
int spm_load_pcm_firmware_nodev(int src)
{
	int i;

	spm_crit("spm_load_pcm_firmware_nodev by src %d\n", src);

	if (spm_fw_count == 0)
		spm_load_pcm_firmware(pspmdev);
	else
		spm_crit("spm_fw_count = %d\n", spm_fw_count);

	for (i = DYNA_LOAD_PCM_SUSPEND; i < DYNA_LOAD_PCM_MAX; i++) {
		struct pcm_desc *pdesc = &(dyna_load_pcm[i].desc);

		if (dyna_load_pcm[i].ready &&
		    (pdesc->addr_2nd == 0) &&
		    (dyna_load_pcm_addr_2nd == (pdesc->size - 3))) {
			spm_crit("recover addr_2nd (%d), %d %d\n", i, dyna_load_pcm_addr_2nd, pdesc->size);
			*(u16 *) &pdesc->size = dyna_load_pcm_addr_2nd;
		}
	}

	return 0;
}
~~~

​	由以上分析应该可以明确：真正load firmware的是在函数**spm_load_pcm_firmware()**中，但是怎么才会调用到这里，还需要一步一步往上探寻，所以这里就先一步一步追踪，把整个流程理清楚。

​	再往上搜索，在**SPM_detect_open()**函数中调用了**spm_load_pcm_firmware_nodev()**

~~~c
static int SPM_detect_open(struct inode *inode, struct file *file)
{
	pr_debug("open major %d minor %d (pid %d)\n", imajor(inode), iminor(inode), current->pid);
	if (dyna_load_pcm_progress == 0)
		spm_load_pcm_firmware_nodev(LOAD_FW_BY_DEV);

	return 0;
}
~~~

​	看到这个函数有static修饰，说明只在本文件下调用，再搜索下，在一个file_operations结构体中调用：

~~~c
const struct file_operations gSPMDetectFops = {
	.open = SPM_detect_open,
	.release = SPM_detect_close,
	.read = SPM_detect_read,
	.write = SPM_detect_write,
};
~~~

​	看起来是一个驱动，上层调用open一个设备就会调用到相应的函数，因此在当前文件下，搜索下**gSPMDetectFops**这个结构体，在函数**spm_module_late_init()**中调用：

~~~c
int spm_module_late_init(void)
{
	int i = 0;
	dev_t devID = MKDEV(gSPMDetectMajor, 0);
	int cdevErr = -1;
	int ret = -1;

	pspmdev = platform_device_register_simple("spm", 0, NULL, 0);
	if (IS_ERR(pspmdev)) {
		pr_debug("Failed to register platform device.\n");
		return -EINVAL;
	}

	ret = register_chrdev_region(devID, SPM_DETECT_DEV_NUM, SPM_DETECT_DRVIER_NAME);
	if (ret) {
		pr_debug("fail to register chrdev\n");
		return ret;
	}

	cdev_init(&gSPMDetectCdev, &gSPMDetectFops);
	gSPMDetectCdev.owner = THIS_MODULE;

	cdevErr = cdev_add(&gSPMDetectCdev, devID, SPM_DETECT_DEV_NUM);
	if (cdevErr) {
		pr_debug("cdev_add() fails (%d)\n", cdevErr);
		goto err1;
	}
    ....
}
~~~

​	可以得知，是创建设备节点，应该是/dev/spm,也就是说，只要open了这个设备节点，整个流程就会如下：

`open("/dev/spm") -> SPM_detect_open() -> spm_load_pcm_firmware_nodev() -> spm_load_pcm_firmware()`

在load了firmware之后，`dyna_load_pcm_done`被置为1，所以会进入sleep。

因此接下来的关键就是，函数**spm_load_pcm_firmware()**是怎么load firmware的。

#### 2.spm_load_pcm_firmware函数load firmware

先查看这个函数：

~~~c
int spm_load_pcm_firmware(struct platform_device *pdev)
{
	const struct firmware *fw;
	int err = 0;
	int i;
	int offset = 0;
	int addr_2nd = 0;

	if (!pdev)
		return err;

	if (dyna_load_pcm_done)
		return err;

	if (local_buf == NULL) {
		local_buf = (char *)ioremap_nocache(local_buf_dma, PCM_FIRMWARE_SIZE * DYNA_LOAD_PCM_MAX);
		if (!local_buf) {
			pr_debug("Failed to dma_alloc_coherent(), %d.\n", err);
			return -ENOMEM;
		}
	}

	for (i = DYNA_LOAD_PCM_SUSPEND; i < DYNA_LOAD_PCM_MAX; i++) {
		u16 firmware_size = 0;
		int copy_size = 0;
		struct pcm_desc *pdesc = &(dyna_load_pcm[i].desc);
		int j = 0;

		spm_fw[i] = NULL;
		do {
			j++;
			pr_debug("try to request_firmware() %s - %d\n", dyna_load_pcm_path[i], j);
			err = request_firmware(&fw, dyna_load_pcm_path[i], &pdev->dev);
			if (err)
				pr_err("Failed to load %s, err = %d.\n", dyna_load_pcm_path[i], err);
		} while (err == -EAGAIN && j < 5);
		if (err) {
			pr_err("Failed to load %s, err = %d.\n", dyna_load_pcm_path[i], err);
			continue;
		}
		spm_fw[i] = fw;

		/* Do whatever it takes to load firmware into device. */
		/* start of binary size */
		offset = 0;
		copy_size = 2;
		memcpy(&firmware_size, fw->data, copy_size);

		/* start of binary */
		offset += copy_size;
		copy_size = firmware_size * 4;
		dyna_load_pcm[i].buf = local_buf + i * PCM_FIRMWARE_SIZE;
		dyna_load_pcm[i].buf_dma = local_buf_dma + i * PCM_FIRMWARE_SIZE;
		memcpy_toio(dyna_load_pcm[i].buf, fw->data + offset, copy_size);
		/* dmac_map_area((void *)dyna_load_pcm[i].buf, PCM_FIRMWARE_SIZE, DMA_TO_DEVICE); */

		/* start of pcm_desc without pointer */
		offset += copy_size;
		copy_size = sizeof(struct pcm_desc) - offsetof(struct pcm_desc, size);
		memcpy((void *)&(dyna_load_pcm[i].desc.size), fw->data + offset, copy_size);
		/* get minimum addr_2nd */
		if (pdesc->addr_2nd) {
			if (addr_2nd)
				addr_2nd = min_t(int, (int)pdesc->addr_2nd, (int)addr_2nd);
			else
				addr_2nd = pdesc->addr_2nd;
		}

		/* start of pcm_desc version */
		offset += copy_size;
		copy_size = fw->size - offset;
		snprintf(dyna_load_pcm[i].version, PCM_FIRMWARE_VERSION_SIZE - 1,
				"%s", fw->data + offset);
		pdesc->version = dyna_load_pcm[i].version;
		pdesc->base = (u32 *) dyna_load_pcm[i].buf;
		pdesc->base_dma = dyna_load_pcm[i].buf_dma;

		dyna_load_pcm[i].ready = 1;
		spm_fw_count++;
	}

	dyna_load_pcm_addr_2nd = addr_2nd;

#if defined(CONFIG_ARCH_MT6755) || defined(CONFIG_MACH_MT6757) || defined(CONFIG_MACH_KIBOPLUS)
	/* check addr_2nd */
	if (spm_fw_count == DYNA_LOAD_PCM_MAX) {
		for (i = DYNA_LOAD_PCM_SUSPEND; i < DYNA_LOAD_PCM_MAX; i++) {
			struct pcm_desc *pdesc = &(dyna_load_pcm[i].desc);

			if (!pdesc->version)
				continue;

			if (pdesc->addr_2nd == 0) {
				if (addr_2nd == (pdesc->size - 3)) {
					*(u16 *) &pdesc->size = addr_2nd;
				} else {
					pr_err("check addr_2nd fail, %d %d\n", addr_2nd, pdesc->size);
					WARN_ON(1);
				}
			}
		}
	}
#endif
	if (spm_fw_count == DYNA_LOAD_PCM_MAX) {
#if defined(CONFIG_MACH_MT6757) || defined(CONFIG_MACH_KIBOPLUS)
		__spm_pmic_low_iq_mode(0);
#endif
#if defined(CONFIG_ARCH_MT6755) || defined(CONFIG_MACH_MT6757) || defined(CONFIG_MACH_KIBOPLUS)
		vcorefs_late_init_dvfs();
#endif
		dyna_load_pcm_done = 1;
		spm_crit("SPM firmware is ready, dyna_load_pcm_done = %d\n", dyna_load_pcm_done);
	}

	return err;
}
~~~

分析上面函数，可知，有一个for循环，`for (i = DYNA_LOAD_PCM_SUSPEND; i < DYNA_LOAD_PCM_MAX; i++)`,在循环里面，应该是去load firmware，有如下代码：

~~~c
spm_fw[i] = NULL;
do {
    j++;
    pr_debug("try to request_firmware() %s - %d\n", dyna_load_pcm_path[i], j);
    err = request_firmware(&fw, dyna_load_pcm_path[i], &pdev->dev);
    if (err)
        pr_err("Failed to load %s, err = %d.\n", dyna_load_pcm_path[i], err);
} while (err == -EAGAIN && j < 5);
if (err) {
    pr_err("Failed to load %s, err = %d.\n", dyna_load_pcm_path[i], err);
    continue;
}
spm_fw[i] = fw;
~~~

这里的`request_firmware()`就是去load firmware的操作，这里还有一个点就是**dyna_load_pcm_path**,这是一个数组，是firmware的名字，如下：

~~~c
static char *dyna_load_pcm_path[] = {
	[DYNA_LOAD_PCM_SUSPEND] = "pcm_suspend_mt6355.bin",
	[DYNA_LOAD_PCM_SUSPEND_BY_MP1] = "pcm_suspend_by_mp1_mt6355.bin",
	[DYNA_LOAD_PCM_SUSPEND_LPDDR4] = "pcm_suspend_lpddr4_mt6355.bin",
	[DYNA_LOAD_PCM_SUSPEND_LPDDR4_BY_MP1] = "pcm_suspend_lpddr4_by_mp1_mt6355.bin",
	[DYNA_LOAD_PCM_SODI] = "pcm_sodi_ddrdfs_mt6355.bin",
	[DYNA_LOAD_PCM_SODI_BY_MP1] = "pcm_sodi_ddrdfs_by_mp1_mt6355.bin",
	[DYNA_LOAD_PCM_SODI_LPDDR4] = "pcm_sodi_ddrdfs_lpddr4_mt6355.bin",
	[DYNA_LOAD_PCM_SODI_LPDDR4_BY_MP1] = "pcm_sodi_ddrdfs_lpddr4_by_mp1_mt6355.bin",
	[DYNA_LOAD_PCM_DEEPIDLE] = "pcm_deepidle_mt6355.bin",
	[DYNA_LOAD_PCM_DEEPIDLE_BY_MP1] = "pcm_deepidle_by_mp1_mt6355.bin",
	[DYNA_LOAD_PCM_DEEPIDLE_LPDDR4] = "pcm_deepidle_lpddr4_mt6355.bin",
	[DYNA_LOAD_PCM_DEEPIDLE_LPDDR4_BY_MP1] = "pcm_deepidle_lpddr4_by_mp1_mt6355.bin",
	[DYNA_LOAD_PCM_MAX] = "pcm_path_max",
};
~~~

到这里，应该大致可以知道，应该是去load这些bin文件，因此下一步是研究`request_firmware()`函数了。

`request_firmware`函数在文件：`kernel-4.4/drivers/base/firmware_class.c`定义。

这一部分涉及kernel加载firmware部分，因此上网查询了，可以参考下面的博客学习。

[kernel空间加载用户空间fw实现原理](http://kernel.meizu.com/implementation-of-loading-fw-from-userspace.html)

以下部分是我个人的思路以及本次问题的解决：

request_firmware函数会尝试从4个地方load相应的fw：

- 从内核中相应的段中查找是否有符合要求的firmw
- 从cache中查找是否有上次load相应的还没有换出firmware
- 直接利用内核中文件接口读取相应的firmware
- 利用uevent接口load相应的firmware

对于前面两种，没怎么看，第三种其实就是直接去指定的文件夹下查找相关的firmware，在firmware_class.c中有以下定义：

~~~c
/* direct firmware loading support */
static char fw_path_para[256];
static const char * const fw_path[] = {
	fw_path_para,
	"/lib/firmware/updates/" UTS_RELEASE,
	"/lib/firmware/updates",
	"/lib/firmware/" UTS_RELEASE,
	"/lib/firmware"
};
~~~

​	这就是firmware的路径了，firmware的名字在上面的dyna_load_pcm_path中有定义，因此在recovery模式下，只需要把相关的firmware复制到这些路径中的某一个，那么在request_firmware中就可以load firmware了。

​	我的解决方法是，在recovery中新建`/lib/firmware`，然后copy固件过去，主要是在文件`build/core/Makefile`中添加，在make recovery image处：

~~~Makefile
@echo ----- Making recovery image ------
  $(hide) mkdir -p $(TARGET_RECOVERY_OUT)
  $(hide) mkdir -p $(TARGET_RECOVERY_ROOT_OUT)/etc $(TARGET_RECOVERY_ROOT_OUT)/sdcard $(TARGET_RECOVERY_ROOT_OUT)/tmp
  @echo Copying baseline ramdisk...
  $(hide) rsync -a --exclude=etc --exclude=sdcard $(IGNORE_CACHE_LINK) $(TARGET_ROOT_OUT) $(TARGET_RECOVERY_OUT) # "cp -Rf" fails to overwrite broken symlinks on Mac.
  @echo Modifying ramdisk contents...
  $(hide) rm -f $(TARGET_RECOVERY_ROOT_OUT)/init*.rc
  $(if $(filter true,$(TARGET_USERIMAGES_USE_UBIFS)), \
    $(hide) cp -f $(recovery_ubiformat) $(TARGET_RECOVERY_ROOT_OUT)/sbin/ubiformat)
  $(hide) cp -f $(recovery_initrc) $(TARGET_RECOVERY_ROOT_OUT)/
  $(hide) rm -f $(TARGET_RECOVERY_ROOT_OUT)/sepolicy
  $(hide) cp -f $(recovery_sepolicy) $(TARGET_RECOVERY_ROOT_OUT)/sepolicy
  $(hide) cp $(TARGET_ROOT_OUT)/init.recovery.*.rc $(TARGET_RECOVERY_ROOT_OUT)/ || true # Ignore error when the src file doesn't exist.
  $(hide) mkdir -p $(TARGET_RECOVERY_ROOT_OUT)/res
  $(hide) rm -rf $(TARGET_RECOVERY_ROOT_OUT)/res/*
  $(hide) cp -rf $(recovery_resources_common)/* $(TARGET_RECOVERY_ROOT_OUT)/res
  $(hide) cp -f $(recovery_font) $(TARGET_RECOVERY_ROOT_OUT)/res/images/font.png
# 在这里添加
  @echo zzy mkdir...
  mkdir -p $(TARGET_RECOVERY_ROOT_OUT)/lib/firmware
  cp -rf vendor/mediatek/proprietary/hardware/spm/mt6757/*.bin  $(TARGET_RECOVERY_ROOT_OUT)/lib/firmware
~~~

​	至于第四种，使用uevent的方式，其实也和第三种类似，只不过是从内核发给用户层，再去load固件，主要查看`system/core/init/devices.cpp`文件

​	这里主要流程是：

​	`device_init() -> coldboot("/sys/class"); -> do_coldboot() -> handle_device_fd() -> parse_event -> handle_firmware_event() -> process_firmware_event() `

​	分析：

​	在kernel，firmware_class.c中request_firmware()的第四种方式`fw_load_from_user_helper() -> _request_firmware_load()-> kobject_uevent()` ，在这里会通过uevent发送.

在kobject_uevent里面会调用kobject_uevent_env，里面有填充了ACTION等，在devices.cpp会解析。

​	在用户层，收到uevent后，解析后还是在指定的文件夹下去操作。如下：

~~~c
static const char *firmware_dirs[] = { "/custom/etc/firmware",
                                       "/etc/firmware",
                                       "/vendor/firmware",
                                       "/firmware/image" };

static void process_firmware_event(struct uevent *uevent)
{
    ...
    
    for (i = 0; i < ARRAY_SIZE(firmware_dirs); i++) {
        char *file = NULL;
        l = asprintf(&file, "%s/%s", firmware_dirs[i], uevent->firmware);
        if (l == -1)
            goto data_free_out;
        fw_fd = open(file, O_RDONLY|O_CLOEXEC);
        free(file);
        if (fw_fd >= 0) {
            if(!load_firmware(fw_fd, loading_fd, data_fd))
                INFO("firmware: copy success { '%s', '%s' }\n", root, uevent->firmware);
            else
                INFO("firmware: copy failure { '%s', '%s' }\n", root, uevent->firmware);
            break;
        }
    }
    ...
}
~~~

​	如果选用第四种方式，也是在文件夹路径下吧固件放进去，再去load。

参考博文：

1. [kernel 空间加载用户空间fw实现原理](http://kernel.meizu.com/implementation-of-loading-fw-from-userspace.html)
2. [Linux 固件子系统----如何更新固件](https://blog.csdn.net/skywalkzf/article/details/7479132#)
3. [Android手机功耗](https://www.jianshu.com/p/89ecd6e99359)
4. [Linux 休眠自动唤醒](https://jouyouyun.github.io/post/linux-auto-suspend/)