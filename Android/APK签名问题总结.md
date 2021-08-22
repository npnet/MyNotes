## APK 签名问题总结

### 1.将动态库打包进apk

需要先有环境 `source ./build/envsetup.sh;  lunch xxx `,否则没有appt

```shell
appt a TestDram2.apk lib/arm64-v8a/libtestdram.so  lib/arm64-v8a/libserialport.so
```



### 2. apk签名

```shell
java -Djava.library.path=. -jar signapk.jar platform.x509.pem platform.pk8 TestDram2.apk TestDram2_sign.apk
```



### 3. ADB安装apk

卸载旧版本apk

```shell
adb shell pm uninstall -k --user 0 软件包名
```

安装新版本apk

```shell
adb install -d -f ./TestDram2_sign.apk
```



### 4.问题总结

Q1: 

Failure [INSTALL_FAILED_INVALID_APK: Failed to extract native libraries, res=-2]

A1:  

TestDram2的apk源码目录下，在` AndroidManifest.xml`文件内, 

删除`android:extractNativeLibs="false"` 或者添加 `android:extractNativeLibs="true"`。



Q2:  

error: existing attribute extractNativeLibs="true" conflicts with --extract-native-libs="false"

A2:  

在系统源码路径 `build/make/core/android_manifest.mk` 下，修改--extract-native-libs的值为true

```makefile
my_manifest_fixer_flags += --extract-native-libs=true
```

再次编译即可编译通过。



Q3:

dlopen failed: library "libc++.so" not found

A3:

这是安卓缺少libc++.so，该库文件在系统源码目录 `out/target/product/out/target/product/k85v1_64/system/lib64/libc++.so` ,

将libc++.so拷贝至签名目录下，跟随其他库文件一并打包进apk内即可解决该问题。



Q4:

java.lang.NullPointerException: Attempt to invoke virtual method 'int java.io.InputStream.read(byte[])' on a null object reference

A4:

TestDram2源码内使用了设备 /dev/ttyS0 ，再新机器内并查无此设备文件名，更改为有效的设备文件名 /dev/ttyHS0即可解决该问题。

