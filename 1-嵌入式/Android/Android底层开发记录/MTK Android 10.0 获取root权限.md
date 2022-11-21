### MTK  Android  10.0获取 Root 权限

一. 先将seLinux关闭

```c
--- a/system/core/init/selinux.cpp
+++ b/system/core/init/selinux.cpp
@@ -97,6 +97,12 @@ EnforcingStatus StatusFromCmdline() {
 }

 bool IsEnforcing() {

+       //add by  for root
+       #if 1
+       return false;
+       #endif
+       //end
+       if (ALLOW_PERMISSIVE_SELINUX) {
            return StatusFromCmdline() == SELINUX_ENFORCING;
        }
```



二. 其次是检查su是否参与编译

```makefile
--- a/device/mediateksample/tb8768tp1_64_bsp/device.mk
+++ b/device/mediateksample/tb8768tp1_64_bsp/device.mk
@@ -172,3 +172,9 @@ PRODUCT_PACKAGES += FotaOverlay
 endif
 #adupsfota end
 
+##add by for root
+#ifeq ($(strip $(HX_VENDOR_ROOT)), yes)
+       PRODUCT_PACKAGES += su
+#endif
+##end
+
```



三.修改属性ro.adb.secure 和 ro.debuggable
```makefile
--- a/build/make/core/main.mk
+++ b/build/make/core/main.mk
@@ -299,6 +299,13 @@ ifneq (,$(user_variant))
   ADDITIONAL_DEFAULT_PROPERTIES += ro.secure=1
   ADDITIONAL_DEFAULT_PROPERTIES += security.perf_harden=1
 
+  ##add by for root 
+  ifeq ($(strip $(HX_VENDOR_ROOT)),yes)
+    ADDITIONAL_DEFAULT_PROPERTIES += ro.adb.secure=0
+    ADDITIONAL_DEFAULT_PROPERTIES += ro.debuggable=1
+  endif
+  ##end
+  
   ifeq ($(user_variant),user)
     ADDITIONAL_DEFAULT_PROPERTIES += ro.adb.secure=1
   endif
```



四. 初始化修改

```c++
--- a/frameworks/base/core/jni/com_android_internal_os_Zygote.cpp
+++ b/frameworks/base/core/jni/com_android_internal_os_Zygote.cpp
@@ -548,6 +548,8 @@ static void EnableKeepCapabilities(fail_fn_t fail_fn) {
 }
 
 static void DropCapabilitiesBoundingSet(fail_fn_t fail_fn) {
+  //add by for root
+  /*
   for (int i = 0; prctl(PR_CAPBSET_READ, i, 0, 0, 0) >= 0; i++) {;
     if (prctl(PR_CAPBSET_DROP, i, 0, 0, 0) == -1) {
       if (errno == EINVAL) {
@@ -558,6 +560,8 @@ static void DropCapabilitiesBoundingSet(fail_fn_t fail_fn) {
       }
     }
   }
+  */
+  //end
 }
 
 static void SetInheritable(uint64_t inheritable, fail_fn_t fail_fn) {
```



五. 修改system下的相关修改

```
--- a/system/core/adb/Android.bp
+++ b/system/core/adb/Android.bp
@@ -25,7 +25,9 @@ cc_defaults {
         "-Wthread-safety",
         "-Wvla",
         "-DADB_HOST=1",         // overridden by adbd_defaults
-        "-DALLOW_ADBD_ROOT=0",  // overridden by adbd_defaults
+        "-DALLOW_ADBD_ROOT=1",  // overridden by adbd_defaults
+               "-DALLOW_ADBD_DISABLE_VERITY=1",
+               "-DALLOW_ADBD_NO_AUTH=1",
     ],
     cpp_std: "experimental",
 
@@ -82,8 +84,8 @@ cc_defaults {
             cflags: [
                 "-UALLOW_ADBD_ROOT",
                 "-DALLOW_ADBD_ROOT=1",
-                "-DALLOW_ADBD_DISABLE_VERITY",
-                "-DALLOW_ADBD_NO_AUTH",
+                "-DALLOW_ADBD_DISABLE_VERITY=1",
+                "-DALLOW_ADBD_NO_AUTH=1",
             ],
         },
     },
@@ -404,13 +406,13 @@ cc_library {
         "liblog",
     ],
 
-    product_variables: {
-        debuggable: {
+    //product_variables: {
+    //    debuggable: {
             required: [
                 "remount",
             ],
-        },
-    },
+    //    },
+    //},
 
     target: {
         android: {
```

```c++
--- a/system/core/adb/daemon/main.cpp
+++ b/system/core/adb/daemon/main.cpp
@@ -72,6 +72,12 @@ static bool should_drop_capabilities_bounding_set() {
 }
 
 static bool should_drop_privileges() {
+       //add by  for root
+       #if 1
+       return false;
+       #endif
+       //end add by for root
+
     // "adb root" not allowed, always drop privileges.
     if (!ALLOW_ADBD_ROOT && !is_device_unlocked()) return true;
```

```
--- a/system/core/fs_mgr/Android.bp
+++ b/system/core/fs_mgr/Android.bp
@@ -76,7 +76,8 @@ cc_library {
         "libfstab",
     ],
     cppflags: [
-        "-DALLOW_ADBD_DISABLE_VERITY=0",
+        "-DALLOW_ADBD_DISABLE_VERITY=1",
+               "-DALLOW_SKIP_SECURE_CHECK=1"
     ],
     product_variables: {
         debuggable: {
@@ -133,7 +134,7 @@ cc_binary {
         "fs_mgr_remount.cpp",
     ],
     cppflags: [
-        "-DALLOW_ADBD_DISABLE_VERITY=0",
+        "-DALLOW_ADBD_DISABLE_VERITY=1",
     ],
     product_variables: {
         debuggable: {
```

```c++
--- a/system/core/libcutils/fs_config.cpp
+++ b/system/core/libcutils/fs_config.cpp
@@ -197,7 +197,7 @@ static const struct fs_path_config android_files[] = {
     // the following two files are INTENTIONALLY set-uid, but they
     // are NOT included on user builds.
     { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/procmem" },
-    { 04750, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
+    { 06755, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
 
     // the following files have enhanced capabilities and ARE included
     // in user builds.
@@ -219,6 +219,7 @@ static const struct fs_path_config android_files[] = {
     { 00750, AID_ROOT,      AID_SHELL,     0, "init*" },
     { 00755, AID_ROOT,      AID_SHELL,     0, "product/bin/*" },
     { 00750, AID_ROOT,      AID_SHELL,     0, "sbin/*" },
+       { 06755, AID_ROOT,      AID_ROOT,     0, "system/xbin/su" },
     { 00755, AID_ROOT,      AID_SHELL,     0, "system/bin/*" },
     { 00755, AID_ROOT,      AID_SHELL,     0, "system/xbin/*" },
     { 00755, AID_ROOT,      AID_SHELL,     0, "system/apex/*/bin/*" },
```

```
--- a/system/core/rootdir/init.rc
+++ b/system/core/rootdir/init.rc
@@ -12,6 +12,10 @@ import /init.usb.configfs.rc
 import /init.${ro.zygote}.rc
 
 # Cgroups are mounted right before early-init using list from /etc/cgroups.json
+
+# add for  start
+import /init.cdfinger.rc
+# add for end
 on early-init
     # Disable sysrq from keyboard
     write /proc/sys/kernel/sysrq 0
@@ -803,6 +807,11 @@ on property:vold.decrypt=trigger_shutdown_framework
 
 on property:sys.boot_completed=1
     bootchart stop
+    
+#add by for root
+start remount
+#end
+    
     # Setup per_boot directory so other .rc could start to use it on boot_completed
     exec - system system -- /bin/rm -rf /data/per_boot
     mkdir /data/per_boot 0700 system system
@@ -858,3 +867,14 @@ on property:ro.debuggable=1
 service flash_recovery /system/bin/install-recovery.sh
     class main
     oneshot
+#add by for root 
+
+
+
+service remount /system/bin/remount
+       class core
+       oneshot
+    disabled
+    user root
+    group root
+#add end
```

六.编译

​	完成编译后将镜像烧录进开发板，开机启动安卓系统后，通过控制台输入命令 su ，当标识符由$ 变为 # 则表示成功进入root模式