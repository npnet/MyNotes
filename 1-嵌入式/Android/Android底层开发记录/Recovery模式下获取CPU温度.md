## Recovery模式下获取CPU温度

平台：MT6757（P25）

版本：Android7.1.1

​	理论上Android获取CPU温度都是一样的，可以通过shell命令获取到，详细获取可以看《Android系统读取CPU温度.md》这一篇，这里直接是`cat /sys/class/thermal/thermal_zone1/temp`

​	但是我当前的P25中，Recovery模式下是无法敲命令以及adb的，所以一开始选择和Android系统一样的处理方式——使用popen来执行shell命令，读取结果是错误的，因此这种方式并不可行。

​	实质上，Recovery模式下应该直接以操作文件的方式去获取温度，使用open，read函数，具体代码如下：

~~~c
static int read_int(char const* path)
{
    int fd;
    int ret = -1;

    if (path == NULL)
    	return -1;

    fd = open(path, O_RDONLY);
    if (fd >= 0) {
    	char buffer[20];
    	int len = read(fd, buffer, 19);
		if (len < 0) {
            close(fd);
            return -1;
		}

    	if (sscanf(buffer, "%d", &ret))
    	{
    	    close(fd);
    	    // success
    	    return ret;
    	}
    	close(fd);
    	return -1;
    }
    return -errno;
}

static int testdram_readCPUTemp(void) 
{
	int cpuTemp = 0;

	cpuTemp = read_int("/sys/class/thermal/thermal_zone1/temp");

	return cpuTemp;
}
~~~

