### 1-Android系统编写Linux内核驱动程序

一. 进入到kernel/common/drivers目录，新建hello目录：

```powershell
USER-NAME@MACHINE-NAME:~/Android$ cd kernel/common/drivers
USER-NAME@MACHINE-NAME:~/Android/kernel/common/drivers$ mkdir hello
```

 

二. 在hello目录中增加hello.h文件：

```c
#ifndef _HELLO_ANDROID_H_
#define _HELLO_ANDROID_H_

#include <linux/cdev.h>
#include <linux/semaphore.h>

#define HELLO_DEVICE_NODE_NAME  "hello"
#define HELLO_DEVICE_FILE_NAME  "hello"
#define HELLO_DEVICE_PROC_NAME  "hello"
#define HELLO_DEVICE_CLASS_NAME "hello"

struct hello_android_dev {
	int val;
	struct semaphore sem;
	struct cdev dev;
};
#endif
```


​		这个头文件定义了一些字符串常量宏，在后面我们要用到。此外，还定义了一个字符设备结构体hello_android_dev，这个就是我们虚拟的硬件设备了，val成员变量就代表设备里面的寄存器，它的类型为int，sem成员变量是一个信号量，是用同步访问寄存器val的，dev成员变量是一个内嵌的字符设备，这个Linux驱动程序自定义字符设备结构体的标准方法。



 三.在hello目录中增加hello.c文件，这是驱动程序的实现部分。驱动程序的功能主要是向上层提供访问设备的寄存器的值，包括读和写。这里，提供了三种访问设备寄存器的方法，一是通过proc文件系统来访问，二是通过传统的设备文件的方法来访问，三是通过devfs文件系统来访问。下面分段描述该驱动程序的实现。

 		首先是包含必要的头文件和定义三种访问设备的方法：

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/types.h>
#include <linux/fs.h>
#include <linux/proc_fs.h>
#include <linux/device.h>
#include <asm/uaccess.h>

#include "hello.h"

/*主设备和从设备号变量*/
static int hello_major = 0;
static int hello_minor = 0;

/*设备类别和设备变量*/
static struct class* hello_class = NULL;
static struct hello_android_dev* hello_dev = NULL;

/*传统的设备文件操作方法*/
static int hello_open(struct inode* inode, struct file* filp);
static int hello_release(struct inode* inode, struct file* filp);
static ssize_t hello_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos);
static ssize_t hello_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos);

/*设备文件操作方法表*/
static struct file_operations hello_fops = {
	.owner = THIS_MODULE,
	.open = hello_open,
	.release = hello_release,
	.read = hello_read,
	.write = hello_write, 
};

/*访问设置属性方法*/
static ssize_t hello_val_show(struct device* dev, struct device_attribute* attr,  char* buf);
static ssize_t hello_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count);

/*定义设备属性*/
static DEVICE_ATTR(val, S_IRUGO | S_IWUSR, hello_val_show, hello_val_store);
```


​        定义传统的设备文件访问方法，主要是定义hello_open、hello_release、hello_read和hello_write这四个打开、释放、读和写设备文件的方法：

```c
/*打开设备方法*/
static int hello_open(struct inode* inode, struct file* filp) {
	struct hello_android_dev* dev;        
	

	/*将自定义设备结构体保存在文件指针的私有数据域中，以便访问设备时拿来用*/
	dev = container_of(inode->i_cdev, struct hello_android_dev, dev);
	filp->private_data = dev;
	
	return 0;

}

/*设备文件释放时调用，空实现*/
static int hello_release(struct inode* inode, struct file* filp) {
	return 0;
}

/*读取设备的寄存器val的值*/
static ssize_t hello_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos) {
	ssize_t err = 0;
	struct hello_android_dev* dev = filp->private_data;        

	/*同步访问*/
	if(down_interruptible(&(dev->sem))) {
		return -ERESTARTSYS;
	}
	 
	if(count < sizeof(dev->val)) {
		goto out;
	}        
	 
	/*将寄存器val的值拷贝到用户提供的缓冲区*/
	if(copy_to_user(buf, &(dev->val), sizeof(dev->val))) {
		err = -EFAULT;
		goto out;
	}
	 
	err = sizeof(dev->val);

out:
	up(&(dev->sem));
	return err;
}

/*写设备的寄存器值val*/
static ssize_t hello_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos) {
	struct hello_android_dev* dev = filp->private_data;
	ssize_t err = 0;        

	/*同步访问*/
	if(down_interruptible(&(dev->sem))) {
		return -ERESTARTSYS;        
	}        
	 
	if(count != sizeof(dev->val)) {
		goto out;        
	}        
	 
	/*将用户提供的缓冲区的值写到设备寄存器去*/
	if(copy_from_user(&(dev->val), buf, count)) {
		err = -EFAULT;
		goto out;
	}
	 
	err = sizeof(dev->val);

out:
	up(&(dev->sem));
	return err;
}
```


​        定义通过devfs文件系统访问方法，这里把设备的寄存器val看成是设备的一个属性，通过读写这个属性来对设备进行访问，主要是实现hello_val_show和hello_val_store两个方法，同时定义了两个内部使用的访问val值的方法__hello_get_val和__hello_set_val：

```c
/*读取寄存器val的值到缓冲区buf中，内部使用*/
static ssize_t __hello_get_val(struct hello_android_dev* dev, char* buf) {
	int val = 0;        

	/*同步访问*/
	if(down_interruptible(&(dev->sem))) {                
		return -ERESTARTSYS;        
	}        
	 
	val = dev->val;        
	up(&(dev->sem));        
	 
	return snprintf(buf, PAGE_SIZE, "%d\n", val);

}

/*把缓冲区buf的值写到设备寄存器val中去，内部使用*/
static ssize_t __hello_set_val(struct hello_android_dev* dev, const char* buf, size_t count) {
	int val = 0;        

	/*将字符串转换成数字*/        
	val = simple_strtol(buf, NULL, 10);        
	 
	/*同步访问*/        
	if(down_interruptible(&(dev->sem))) {                
		return -ERESTARTSYS;        
	}        
	 
	dev->val = val;        
	up(&(dev->sem));
	 
	return count;

}

/*读取设备属性val*/
static ssize_t hello_val_show(struct device* dev, struct device_attribute* attr, char* buf) {
	struct hello_android_dev* hdev = (struct hello_android_dev*)dev_get_drvdata(dev);        

	return __hello_get_val(hdev, buf);

}

/*写设备属性val*/
static ssize_t hello_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count) { 
	struct hello_android_dev* hdev = (struct hello_android_dev*)dev_get_drvdata(dev);  
	

	return __hello_set_val(hdev, buf, count);

}
```


​        定义通过proc文件系统访问方法，主要实现了hello_proc_read和hello_proc_write两个方法，同时定义了在proc文件系统创建和删除文件的方法hello_create_proc和hello_remove_proc：

```c
/*读取设备寄存器val的值，保存在page缓冲区中*/
static ssize_t hello_proc_read(char* page, char** start, off_t off, int count, int* eof, void* data) {
	if(off > 0) {
		*eof = 1;
		return 0;
	}

	return __hello_get_val(hello_dev, page);

}

/*把缓冲区的值buff保存到设备寄存器val中去*/
static ssize_t hello_proc_write(struct file* filp, const char __user *buff, unsigned long len, void* data) {
	int err = 0;
	char* page = NULL;

	if(len > PAGE_SIZE) {
		printk(KERN_ALERT"The buff is too large: %lu.\n", len);
		return -EFAULT;
	}
	 
	page = (char*)__get_free_page(GFP_KERNEL);
	if(!page) {                
		printk(KERN_ALERT"Failed to alloc page.\n");
		return -ENOMEM;
	}        
	 
	/*先把用户提供的缓冲区值拷贝到内核缓冲区中去*/
	if(copy_from_user(page, buff, len)) {
		printk(KERN_ALERT"Failed to copy buff from user.\n");                
		err = -EFAULT;
		goto out;
	}
	 
	err = __hello_set_val(hello_dev, page, len);

out:
	free_page((unsigned long)page);
	return err;
}

/*创建/proc/hello文件*/
static void hello_create_proc(void) {
	struct proc_dir_entry* entry;
	

	entry = create_proc_entry(HELLO_DEVICE_PROC_NAME, 0, NULL);
	if(entry) {
		entry->owner = THIS_MODULE;
		entry->read_proc = hello_proc_read;
		entry->write_proc = hello_proc_write;
	}

}

/*删除/proc/hello文件*/
static void hello_remove_proc(void) {
	remove_proc_entry(HELLO_DEVICE_PROC_NAME, NULL);
}
   最后，定义模块加载和卸载方法，这里只要是执行设备注册和初始化操作：


/*初始化设备*/
static int  __hello_setup_dev(struct hello_android_dev* dev) {
	int err;
	dev_t devno = MKDEV(hello_major, hello_minor);

	memset(dev, 0, sizeof(struct hello_android_dev));
	 
	cdev_init(&(dev->dev), &hello_fops);
	dev->dev.owner = THIS_MODULE;
	dev->dev.ops = &hello_fops;        
	 
	/*注册字符设备*/
	err = cdev_add(&(dev->dev),devno, 1);
	if(err) {
		return err;
	}        
	 
	/*初始化信号量和寄存器val的值*/
	init_MUTEX(&(dev->sem));
	dev->val = 0;
	 
	return 0;

}

/*模块加载方法*/
static int __init hello_init(void){ 
	int err = -1;
	dev_t dev = 0;
	struct device* temp = NULL;

	printk(KERN_ALERT"Initializing hello device.\n");        
	 
	/*动态分配主设备和从设备号*/
	err = alloc_chrdev_region(&dev, 0, 1, HELLO_DEVICE_NODE_NAME);
	if(err < 0) {
		printk(KERN_ALERT"Failed to alloc char dev region.\n");
		goto fail;
	}
	 
	hello_major = MAJOR(dev);
	hello_minor = MINOR(dev);        
	 
	/*分配helo设备结构体变量*/
	hello_dev = kmalloc(sizeof(struct hello_android_dev), GFP_KERNEL);
	if(!hello_dev) {
		err = -ENOMEM;
		printk(KERN_ALERT"Failed to alloc hello_dev.\n");
		goto unregister;
	}        
	 
	/*初始化设备*/
	err = __hello_setup_dev(hello_dev);
	if(err) {
		printk(KERN_ALERT"Failed to setup dev: %d.\n", err);
		goto cleanup;
	}        
	 
	/*在/sys/class/目录下创建设备类别目录hello*/
	hello_class = class_create(THIS_MODULE, HELLO_DEVICE_CLASS_NAME);
	if(IS_ERR(hello_class)) {
		err = PTR_ERR(hello_class);
		printk(KERN_ALERT"Failed to create hello class.\n");
		goto destroy_cdev;
	}        
	 
	/*在/dev/目录和/sys/class/hello目录下分别创建设备文件hello*/
	temp = device_create(hello_class, NULL, dev, "%s", HELLO_DEVICE_FILE_NAME);
	if(IS_ERR(temp)) {
		err = PTR_ERR(temp);
		printk(KERN_ALERT"Failed to create hello device.");
		goto destroy_class;
	}        
	 
	/*在/sys/class/hello/hello目录下创建属性文件val*/
	err = device_create_file(temp, &dev_attr_val);
	if(err < 0) {
		printk(KERN_ALERT"Failed to create attribute val.");                
		goto destroy_device;
	}
	 
	dev_set_drvdata(temp, hello_dev);        
	 
	/*创建/proc/hello文件*/
	hello_create_proc();
	 
	printk(KERN_ALERT"Succedded to initialize hello device.\n");
	return 0;

destroy_device:
	device_destroy(hello_class, dev);

destroy_class:
	class_destroy(hello_class);

destroy_cdev:
	cdev_del(&(hello_dev->dev));

cleanup:
	kfree(hello_dev);

unregister:
	unregister_chrdev_region(MKDEV(hello_major, hello_minor), 1);

fail:
	return err;
}

/*模块卸载方法*/
static void __exit hello_exit(void) {
	dev_t devno = MKDEV(hello_major, hello_minor);

	printk(KERN_ALERT"Destroy hello device.\n");        
	 
	/*删除/proc/hello文件*/
	hello_remove_proc();        
	 
	/*销毁设备类别和设备*/
	if(hello_class) {
		device_destroy(hello_class, MKDEV(hello_major, hello_minor));
		class_destroy(hello_class);
	}        
	 
	/*删除字符设备和释放设备内存*/
	if(hello_dev) {
		cdev_del(&(hello_dev->dev));
		kfree(hello_dev);
	}        
	 
	/*释放设备号*/
	unregister_chrdev_region(devno, 1);

}

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("First Android Driver");

module_init(hello_init);
module_exit(hello_exit);
```



五.在hello目录中新增Kconfig和Makefile两个文件，其中Kconfig是在编译前执行配置命令make menuconfig时用到的，而Makefile是执行编译命令make是用到的：

Kconfig文件的内容

```makefile
config HELLO
    tristate "First Android Driver"
    default n
    help
    This is the first android driver.
 
```
 Makefile文件的内容

```
 obj-$(CONFIG_HELLO) += hello.o
```

  		在Kconfig文件中，tristate表示编译选项HELLO支持在编译内核时，hello模块支持以模块、内建和不编译三种编译方法，默认是不编译，因此，在编译内核前，我们还需要执行make menuconfig命令来配置编译选项，使得hello可以以模块或者内建的方法进行编译。
在Makefile文件中，根据选项HELLO的值，执行不同的编译方法。



六. 修改arch/arm/Kconfig和drivers/kconfig两个文件，在menu "Device Drivers"和endmenu之间添加一行：

```
 source "drivers/hello/Kconfig"
```

​    这样，执行make menuconfig时，就可以配置hello模块的编译选项了。. 



七. 修改drivers/Makefile文件，添加一行：

```
obj-$(CONFIG_HELLO) += hello/
```

​    

八. 配置编译选项：
     USER-NAME@MACHINE-NAME:~/Android/kernel/common$ make menuconfig
    找到"Device Drivers" => "First Android Drivers"选项，设置为y。
    注意，如果内核不支持动态加载模块，这里不能选择m，虽然我们在Kconfig文件中配置了HELLO选项为tristate。要支持动态加载模块选项，必须要在配置菜单中选择Enable loadable module support选项；在支持动态卸载模块选项，必须要在Enable loadable module support菜单项中，选择Module unloading选项。



九. 编译：
     USER-NAME@MACHINE-NAME:~/Android/kernel/common$ make
    编译成功后，就可以在hello目录下看到hello.o文件了，这时候编译出来的zImage已经包含了hello驱动。
    

十. 参照 在Ubuntu上下载、编译和安装Android最新内核源代码（Linux Kernel）一文所示，运行新编译的内核文件，验证hello驱动程序是否已经正常安装：

​    进入到dev目录，可以看到hello设备文件：
​    root@android:/ # cd dev
​    root@android:/dev # ls

​    进入到proc目录，可以看到hello文件：
​    root@android:/ # cd proc
​    root@android:/proc # ls

​    访问hello文件的值：
​    root@android:/proc # cat hello
​    0
​    root@android:/proc # echo '5' > hello
​    root@android:/proc # cat hello
​    5

​    进入到sys/class目录，可以看到hello目录：
​    root@android:/ # cd sys/class
​    root@android:/sys/class # ls

​    进入到hello目录，可以看到hello目录：
​    root@android:/sys/class # cd hello
​    root@android:/sys/class/hello # ls

​    进入到下一层hello目录，可以看到val文件：
​    root@android:/sys/class/hello # cd hello
​    root@android:/sys/class/hello/hello # ls

​    访问属性文件val的值：
​    root@android:/sys/class/hello/hello # cat val
​    5
​    root@android:/sys/class/hello/hello # echo '0'  > val
​    root@android:/sys/class/hello/hello # cat val
​    0