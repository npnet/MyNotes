## Android限制kernel的mem=128M

#### 目的：

在recovery测试时，分段测试，kernel分了两段，比如1GB的reserved地址如下：

```
0x40000000--0x48000000 0x08000000 kernel	// 128M
0x48000000--0x7A000000 0x32000000 reserve
0x7A000000--0x80000000 0x06000000 kernel 
```

希望把kernel的reserved 内存限制在128M以内，这样recovery就可以直接测试后面的一整片地址，前128M留给PL去测试。

#### 操作：

1.修改LK

- 主要文件

  新增一个文件：**memtest_recovery_128M.c**

  添加内容：

  ~~~c
  void memtest_recovery_128M(void)
  {
  	cmdline_append("mem=128M");
  	printf("zzy mem = 128M\n");
  }
  ~~~

- 作用

  在PL->LK->kernel的命令参数传递中，将mem=128M传递给kernel，限制kernel的内存大小为128M。

2.修改kernel

- 主要文件

  **.config**文件，路径：`out/target/product/tk6757_66_n1/obj/KERNEL_OBJ/`

- 操作

  kernel的操作不是直接修改.config文件，而是通过menuconfig来修改内核配置。

  打开menuconfig的方法是：在根目录下敲入以下命令

  `make -C kernel-4.4 O=$OUT/obj/KERNEL_OBJ ARCH=arm64 menuconfig`

  然后再进行配置。

  主要裁剪部分是：LCD，FB，GPS，I2C

3.遇到的问题

在裁剪完后，编译OK，但是在测试后面的地址时，出现test pass 但会死机然后重启的现象。

针对这个问题，解决思路是这样的：

首先，先将串口log全部由串口1输出，PL的log在另一个串口，PL中修改如下

文件：**cust_bldr.mak**

~~~
CFG_UART_LOG :=UART1	// 这里之前是UART2
CFG_UART_META :=UART2
CFG_TEE_SUPPORT = 0
CFG_TRUSTONIC_TEE_SUPPORT = 0
CFG_MICROTRUST_TEE_SUPPORT = 0
CFG_LONGSYS_MEM_TEST := 0	// 这里之前是1
~~~

然后抓取log，看看相应地址是不是被release了，我这次测试刚好是抓到地址**0X7FF400000**开始的，大小**0x40000**的地方就卡死，所以我就找下这个地址，有如下：

~~~txt
// LK阶段
[1540] mblock_reserve dbg[1]: 0, 1, 1, 1
[1540] mblock_reserve: 7e360000 - 7ff40000 from mblock 1
[1540] mblock_reserve [0].start: 0x40000000, sz: 0x20000000
[1540] mblock_reserve [1].start: 0x60000000, sz: 0x1e360000
[1540] mblock_reserve-R[0].start: 0x7ff80000, sz: 0x80000 map:0 name:log_store_init
[1540] mblock_reserve-R[1].start: 0x7ff40000, sz: 0x40000 map:0 name:trustzone_pre_init
[1540] mblock_reserve-R[2].start: 0x7e360000, sz: 0x1be0000 map:0 name:platform_init
[1540] FB base = 0x7e360000, FB size = 0x1be0000 (29229056)

[5880] mblock_reserve: 76000000 - 7e000000 from mblock 1
[5880] mblock_reserve [0].start: 0x40000000, sz: 0x20000000
[5880] mblock_reserve [1].start: 0x60000000, sz: 0x16000000
[5880] mblock_reserve [2].start: 0x7e000000, sz: 0x360000
[5880] mblock_reserve-R[0].start: 0x7ff80000, sz: 0x80000 map:0 name:log_store_init
[5880] mblock_reserve-R[1].start: 0x7ff40000, sz: 0x40000 map:0 name:trustzone_pre_init
[5880] mblock_reserve-R[2].start: 0x7e360000, sz: 0x1be0000 map:0 name:platform_init
[5880] mblock_reserve-R[3].start: 0x76000000, sz: 0x8000000 map:0 name:ccci_request_mem
[5880] request size: 0x08000000, get start address: 0x76000000


[6140] mblock_reserved_append: add_subnode name=mblock-1-log_store_init start:0x7ff80000 size:0x80000
[6160] mblock-reserved-memory is appended (0x7ff80000, 0x80000)
[6160] mblock_reserved_append: add_subnode name=mblock-2-trustzone_pre_init start:0x7ff40000 size:0x40000
[6160] mblock-reserved-memory is appended (0x7ff40000, 0x40000)
[6160] mblock_reserved_append: add_subnode name=mblock-3-platform_init start:0x7e360000 size:0x1be0000

// kernel 阶段
[    1.201250] <5>.(4)[1:swapper/0]ATF reserved memory: 0x7ff40000 - 0x7ff7ffff (0x40000)
[    1.201270] <5>.(4)[1:swapper/0]atf_buf_phy_ctl: 0x7ff40000
~~~

发现地址0x7ff40000有这么几个地方有，发现有被reserved了，然后一步一步往回找，首先查找LK：

因为看到有**mblock-reserved_append**,猜测这里是有添加了reserved，到代码中查找相关，找到文件：`alps-p25/vendor/mediatek/proprietary/bootable/bootloader/lk/lib/mblock/mblock_v2.c`

函数：**mblock_reserved_append(void *fdt)**

~~~c
int mblock_reserved_append(void *fdt)
{
	int offset, ret = 0;
	int nodeoffset = 0;
	unsigned int i;
	char node_name[128];
	char compatible[MBLOCK_RESERVED_NAME_SIZE + 64];
	dt_dram_info reg_info;
	reserved_t *reserved;
 
	offset = fdt_path_offset(fdt, "/reserved-memory");
	for (i = 0; i < g_boot_arg->mblock_info.reserved_num; i++) {
		reserved = &g_boot_arg->mblock_info.reserved[i];
		snprintf(node_name, sizeof(node_name), "mblock-%d-%s", i+1, reserved->name);
		dprintf(INFO, "%s: add_subnode name=%s start:0x%llx size:0x%llx\n", __func__, node_name, reserved->start, reserved->size);
		nodeoffset = fdt_add_subnode(fdt, offset, node_name);
		if (nodeoffset < 0) {
			dprintf(CRITICAL, "Warning: can't add mblock-reserved-memory node in device tree nodeoffset=0x%x\n", nodeoffset);
			return 1;
		}
		snprintf(compatible, sizeof(compatible), "mediatek,%s", reserved->name);
		ret = fdt_setprop_string(fdt, nodeoffset, "compatible", compatible);
		if (ret) {
			dprintf(CRITICAL, "Warning: can't add mblock-reserved-memory compatible property in device tree ret=0x%x\n", ret);
			return 1;
		}
		if (!reserved->mapping) {
			ret = fdt_setprop(fdt, nodeoffset, "no-map", NULL, 0);
			if (ret) {
				dprintf(CRITICAL, "Warning: can't add mblock-reserved-memory no-map property in device tree ret=0x%x\n", ret);
				return 1;
			}
		}

		reg_info.start_hi = cpu_to_fdt32(reserved->start>>32);
		reg_info.start_lo = cpu_to_fdt32(reserved->start);
		reg_info.size_hi = cpu_to_fdt32((reserved->size)>>32);
		reg_info.size_lo = cpu_to_fdt32(reserved->size);

		ret = fdt_setprop(fdt, nodeoffset, "reg", &reg_info, sizeof(reg_info));
		if (ret) {
			dprintf(CRITICAL, "Warning: can't add mblock-reserved-memory reg property in device tree ret=0x%x\n", ret);
			return 1;
		}
		dprintf(CRITICAL, "mblock-reserved-memory is appended (0x%llx, 0x%llx)\n",
				reserved->start, reserved->size);
	}
	return ret;
}
~~~

通过阅读函数代码，明白，这个是往dts中添加node，所以猜测这个reserved的不是dts里限制了，而是代码里限制了，进一步查找代码。

**关键在这里：`reserved = &g_boot_arg->mblock_info.reserved[i];`**

继续分析：看到g_boot_arg，应该可以猜测是boot参数，查找代码，应该是从PL传递过来，代码跳转到PL中搜索mblock_info。

在PL中，找到文件：`platform/mt6757/src/security/trustzone/tz_init.c`

有两处地方有**bootarg.mblock_info**

第一处在函数：**tee_get_load_addr(u32 maddr)**

~~~C
u32 tee_get_load_addr(u32 maddr)
{
    u32 ret_addr = 0;
    u64 limit_addr = 0xC0000000;

#if CFG_TEE_SUPPORT
    if (tee_secmem_start != 0)
        goto allocated;

    tee_extra_mem_size = maddr % TEE_MEM_ALIGNMENT;
    tee_secmem_size = maddr - tee_extra_mem_size;

#if CFG_MICROTRUST_TEE_SUPPORT
    limit_addr = 0x80000000;
#endif
#if CFG_GOOGLE_TRUSTY_SUPPORT
    tee_secmem_start = TRUSTY_MEM_LOAD_ADDR;
#else
    tee_secmem_start = (u32)mblock_reserve(&bootarg.mblock_info,
        (u64)(tee_secmem_size + ATF_LOG_BUFFER_SIZE), (u64)TEE_MEM_ALIGNMENT,
        limit_addr, RANKMAX);
#endif // !CFG_GOOGLE_TRUSTY_SUPPORT

    if(!tee_secmem_start){
        printf("%s Fail to allocate secure memory: 0x%x, 0x%x\n", MOD,
            (tee_secmem_size + ATF_LOG_BUFFER_SIZE), TEE_MEM_ALIGNMENT);
        return 0;
    }

    atf_log_buf_start = tee_secmem_start;
    tee_secmem_start = tee_secmem_start + ATF_LOG_BUFFER_SIZE;

allocated:
    ret_addr = tee_secmem_start - tee_extra_mem_size;
#endif /* end of CFG_TEE_SUPPORT */

    return ret_addr;
}
~~~

本来看到这个**limit_addr**,和**mblock_reserve** 大致就可以猜测到了，但可惜的是这里的宏**CFG_TEE_SUPPORT**并没有定义。所以只能看下一处地方了。

第二处在函数：**void trustzone_pre_init(void)** (发现这个函数其实在串口log中出现过)

~~~C
void trustzone_pre_init(void)
{
    // add by zhanye
    u64 limit_addr = 0x48000000;
    
    sec_malloc_buf_reset();

    tz_dapc_sec_init();

#if CFG_ATF_LOG_SUPPORT
#if !CFG_TEE_SUPPORT
    {
        atf_arg_t_ptr teearg = (atf_arg_t_ptr)trustzone_get_atf_boot_param_addr();
        // modify by zhanye
        // atf_log_buf_start = (u32)mblock_reserve(&bootarg.mblock_info,
        //     (u64)ATF_LOG_BUFFER_SIZE, (u64)TEE_MEM_ALIGNMENT,
        //     0xC0000000, RANKMAX);
        atf_log_buf_start = (u32)mblock_reserve(&bootarg.mblock_info,
            (u64)ATF_LOG_BUFFER_SIZE, (u64)TEE_MEM_ALIGNMENT,
            limit_addr, RANKMAX);
        if(!atf_log_buf_start){
            printf("%s Fail to allocate atf log buffer: 0x%x, 0x%x\n", MOD,
                ATF_LOG_BUFFER_SIZE, TEE_MEM_ALIGNMENT);
            teearg->atf_log_buf_size = 0;
        }
        DBG_MSG("zzy atf_log_buf_start = 0x%x\n", atf_log_buf_start);
    }
#endif
#endif
}
~~~

查看了mblock_reserve这个函数的参数之后，知道这里限制了内存最高地址为0XC000000,所以系统分配时候就给分配到了0X7FF40000了，所以就将这里的地址限制为128M的最高地址，0X48000000，这样系统给这个reserved内存时，就会限制在0X48000000以内了。

到此，这个内存限制就完成了，测试也pass了。



