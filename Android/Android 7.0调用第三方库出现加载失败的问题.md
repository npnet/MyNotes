## Android 7.0调用第三方库出现加载失败的问题

### 问题

在Android7.0 ，自己编译的APK放到系统里，调用[第三方库](https://so.csdn.net/so/search?q=第三方库&spm=1001.2101.3001.7020)没有问题，但是通过SD卡点击apk文件安装，就出现

```shell
java.lang.UnsatisfiedLinkError: dlopen failed: library "/system/lib64/libxxx.so"
 needed or dlopened by "/system/lib64/libnativeloader.so" is not accessible for the namespace "classloader-namespace"
```

这个apk可是系统权限的，就是apk的清单AndroidManifest中有下面一句

```shell
 android:sharedUserId="android.uid.system"
```

正常来说，这种高端apk的permitted_paths是包含system/lib64的

如果是系统apk并且没有升级过的话，so库的搜索路径就会增加一个system/lib64。

因为install -r来安装apk就相当于升级，所以刷机时apk可以用，install升级后不能用。



### 解决

应用可以调用`/vendor/etc/public.libraries.txt`和`/system/etc/public.libraries.tx`t里面的所有so库，

所以往这个文件写入自己希望被调用的so，这个库就变成共用的了，任意应用就可以找到这个so库了

MTK平台文件源码路径为 `system/core/rootdir/etc/public.libraries.android.txt`



### 总结

1. Android N 不能直接调用系统的一些私有库了，公用的库都定义在public.libraries.txt里面。

2. 系统应用刚刷机是能够调用system/lib64下的库，但通过install升级该应用时，应用打开会挂。

   因为升级后permitted_paths就不再包含system/lib64了。

   所以我们可以将apk要用到的库名称写到public.libraries.txt中去解决快速调试问题。

