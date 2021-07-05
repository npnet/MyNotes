### Linux下对gpio的控制（基于文件系统）

一.Clean Kernel

```shell
make clean-kernel
```



二. 开启内核配置

- kernel-4.14/arch/arm64/configs/k85v1_64_defconfig

- kernel-4.14/arch/arm64/configs/k85v1_64_debug_defconfig

  添加以下配置

```makefile
CONFIG_GPIO_SYSFS=y
```



三.配置GPIO口信息

编译完成后将镜像烧录进开发板，启动安卓，若出现文件夹/sys/class/gpio，则说明gpio文件系统已开启，可进行GPIO口配置。

进入 /sys/class/gpio 文件夹，显示的gpiochip301，则表示301为GPIO的base num，相应的GPIO4的GPIO num为301+4=305。

1. 创建GPIO4

   ```shell
   echo 305 > /sys/class/gpio/export
   ```

   创建成功会在/sys/class/gpio目录下生成 gpio305文件夹

   

2. 更改为输出模式

   ```shell
   echo out > /sys/class/gpio/gpio305/direction
   ```

   

3. 设置输出低电平

   ```shell
   echo 0 > /sys/class/gpio/gpio305/value
   ```

完成上述配置后，GPIO灯即可正常工作。