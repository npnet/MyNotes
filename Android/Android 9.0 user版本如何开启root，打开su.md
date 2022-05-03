## Android 9.0 user版本如何开启root，打开su

### 1.开启adbd的root的权限

第一种方式

1. /system/core/adb/daemon/main.cpp

```shell
static bool should_drop_privileges() {
  // 注释方法内代码,添加如下代码
  std::string prop = android::base::GetProperty("service.adb.root", "");
  if (prop == "1"){
 	  return false;
  }
  return true;
}
```



第二种方式

1. 修改 /build/core/main.mk

```shell
## user/userdebug ##

user_variant := $(filter user userdebug,$(TARGET_BUILD_VARIANT))
enable_target_debugging := true
tags_to_install :=
ifneq (,$(user_variant))
  # Target is secure in user builds.
  ADDITIONAL_DEFAULT_PROPERTIES += ro.secure=1
  ADDITIONAL_DEFAULT_PROPERTIES += security.perf_harden=1
    ...
将·ro.secure=1 修改为ro.secure=0
```

2. 修改 /system/core/adb/Android.mk

```shell
- ifneq (,$(filter userdebug eng,$(TARGET_BUILD_VARIANT)))
+ ifneq (,$(filter user userdebug eng,$(TARGET_BUILD_VARIANT)))
  LOCAL_CFLAGS += -DALLOW_ADBD_DISABLE_VERITY=1
  LOCAL_CFLAGS += -DALLOW_ADBD_ROOT=1
  endif
```



### 2. 添加su

1. 去掉root，shell的判断

```shell
# /system/extras/su/su.cpp

int main(int argc, char** argv) {
+    #if 0
     uid_t current_uid = getuid();
     if (current_uid != AID_ROOT && current_uid != AID_SHELL) error(1, 0, "not allowed");
+    #endif
     // Handle -h and --help.
```

2. 修改su权限

```shell
# /system/core/rootdir/init.rc

+    chmod 6755 /system/xbin/su
+ 
     # Assume SMP uses shared cpufreq policy for all CPUs
     
# /system/core/libcutils/fs_config.cpp 

-    { 04750, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
+    { 06755, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
```

3. 修改su模块在所有模式下都编译

```shell
# /system/extras/su/Android.mk

- LOCAL_MODULE_TAGS := debug
+ LOCAL_MODULE_TAGS := optional
```

4. 修改 /frameworks/base/core/jni/com_android_internal_os_Zygote.cpp

```shell
static bool DropCapabilitiesBoundingSet(std::string* error_msg) {
+  #if 0
   for (int i = 0; prctl(PR_CAPBSET_READ, i, 0, 0, 0) >= 0; i++) {
    int rc = prctl(PR_CAPBSET_DROP, i, 0, 0, 0);
    if (rc == -1) {
      if (errno == EINVAL) {
        ALOGE("prctl(PR_CAPBSET_DROP) failed with EINVAL. Please verify "
              "your kernel is compiled with file capabilities support");
      } else {
        *error_msg = CREATE_ERROR("prctl(PR_CAPBSET_DROP, %d) failed: %s", i, strerror(errno));
        return false;
      }
    }
  }
+ #endif
   return true;
 }
```

5. 在device.mk下添加su模块

```shell
# /device/mediatek/common/device.mk

PRODUCT_PACKAGES += su
```



### 3.关闭selinux

1. Android 8.0

```shell
 # /system/core/init/init.cpp
 static bool selinux_is_enforcing(void){   
    return false; //add to close selinux
    if (ALLOW_PERMISSIVE_SELINUX) {
        return selinux_status_from_cmdline() == SELINUX_ENFORCING;
    }
    return true;
 }
```

2. Android 9.0

```shell
# /system/core/init/selinux.cpp
 bool IsEnforcing() {
 	return false; //add to close selinux
  	if (ALLOW_PERMISSIVE_SELINUX) {
    	return StatusFromCmdline() == SELINUX_ENFORCING;
   	}
  	return true;
 }
```

