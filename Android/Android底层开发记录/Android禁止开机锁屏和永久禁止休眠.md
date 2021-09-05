## Android 禁止开机锁屏和永久禁止休眠

平台：MTK6757

版本：Android 7.0

### 1. 禁止开机锁屏

修改defaults.xml文件里的**def_lockscreen_disabled**值

文件路径：**alps-p25/frameworks/base/packages/SettingsProvider/res/values/defaults.xml**

修改（约82行）：

```xml
- <bool name="def_lockscreen_disabled">false</bool>
+ <bool name="def_lockscreen_disabled">true</bool>
```

### 2.永久禁止休眠

- 第一步：

  修改defaults.xml文件里的**def_screen_off_timeout**的值

  文件路径：**alps-p25/frameworks/base/packages/SettingsProvider/res/values/defaults.xml**

  修改（约21行）：

  ```xml
  - <integer name="def_screen_off_timeout">60</integer>
  + <integer name="def_screen_off_timeout">-1</integer>
  ```

- 第二步：

  修改PowerManagerService.java文件中的**getScreenOffTimeoutLocked**函数

  路径：**alps-p25/frameworks/base/services/core/java/com/android/server/power/PowerManagerService.java**

  修改（约2102行）：

  ```java
  private int getScreenOffTimeoutLocked(int sleepTimeout) {
  +       int nosleep;
  
          int timeout = mScreenOffTimeoutSetting;
          if (isMaximumScreenOffTimeoutFromDeviceAdminEnforcedLocked()) {
              timeout = Math.min(timeout, mMaximumScreenOffTimeoutFromDeviceAdmin);
          }
          if (mUserActivityTimeoutOverrideFromWindowManager >= 0) {
              timeout = (int)Math.min(timeout, mUserActivityTimeoutOverrideFromWindowManager);
          }
  +        // add by zhanye
  +        nosleep = mScreenOffTimeoutSetting; 
  +        if(nosleep < 0){
  +            nosleep = mMaximumScreenOffTimeoutFromDeviceAdmin;
  +            return Math.max(nosleep, mMaximumScreenOffTimeoutFromDeviceAdmin);
  +        }
  +        /*
          if (sleepTimeout >= 0) {
              timeout = Math.min(timeout, sleepTimeout);
          }
          if (mUserActivityTimeoutMin) {
              timeout = (int)Math.min(timeout, mUserActivityTimeoutOverrideFromCMD);
          }
  +        */
          return Math.max(timeout, mMinimumScreenOffTimeoutConfig);
      }
  ```




















