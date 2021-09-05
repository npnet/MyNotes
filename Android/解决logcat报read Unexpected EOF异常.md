## 解决logcat报read: Unexpected EOF!异常

### 1.扩大缓存区大小命令

```shell
logcat -G 4M
```



### 2. 手动修改系统设置

打开手机调试设备，依次点击 `开发者选项` -->  `日志记录器缓冲区大小`，即可修改缓存区大小。

系统默认的缓存区大小为256KB，可自行设置超过默认大小的值。



### 3. 查看 buffer size 命令

查看当前设备缓冲区大小，在终端输入：

```shell
logcat -g
```

```shell
console:/ # logcat -g                                                      
main: ring buffer is 256 KiB (242 KiB consumed), max entry is 5120 B, max payload is 4068 B
system: ring buffer is 256 KiB (239 KiB consumed), max entry is 5120 B, max payload is 4068 B
crash: ring buffer is 256 KiB (0 B consumed), max entry is 5120 B, max payload is 4068 B
```

