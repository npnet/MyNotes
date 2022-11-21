### 安卓开机自启动服务运行shell脚本

一.编写脚本文件gpio_led.sh

- device/mediatek/mt6771/gpio_led.sh

```shell
#!/system/bin/sh
# Create GPIO4/5/6
echo 324 > /sys/class/gpio/export
echo 325 > /sys/class/gpio/export
echo 326 > /sys/class/gpio/export

# Change GPIO direction and value mode
chmod 777 /sys/class/gpio/gpio324/direction
chmod 777 /sys/class/gpio/gpio325/direction
chmod 777 /sys/class/gpio/gpio326/direction

chmod 777 /sys/class/gpio/gpio324/value
chmod 777 /sys/class/gpio/gpio325/value
chmod 777 /sys/class/gpio/gpio326/value

# Set output direction
echo out > /sys/class/gpio/gpio324/direction
echo out > /sys/class/gpio/gpio325/direction
echo out > /sys/class/gpio/gpio326/direction

# Set high value 
echo 1 > /sys/class/gpio/gpio324/value
echo 1 > /sys/class/gpio/gpio325/value
echo 1 > /sys/class/gpio/gpio326/value

exit 1
```



二.配置开机启动shell脚本

- device/mediatek/mt6771/device.mk

```makefile
PRODUCT_COPY_FILES += device/mediatek/mt6771/gpio_led.sh:system/bin/gpio_led.sh
```

- system/core/rootdir/init.rc

```
service gpio_led /system/bin/gpio_led.sh
	user root
	group root
	oneshot
	disabled
	
on property:sys.boot_completed=1
	start gpio_led
```



三.增加所需权限

- device/mediatek/mt6771/sepolicy/basic/gpio_led.te

```
# ===========================================
# gpio_led sh selinux
# ===========================================
type gpio_led, domain, coredomain;
type gpio_led_exec, exec_type, file_type;
init_daemon_domain(gpio_led)
```

- device/mediatek/mt6771/sepolicy/basic/file_contexts

```
/system/bin/gpio_led.sh u:object_r:gpio_led_exec:s0
```

通过 init 启动的服务需要在各自的 selinux 域中运行。需完成权限配置以便 selinux 在适当的网域中运行相应服务。