## Android 设置默认打开usb debug

平台：MT6757

版本：Android7.0

### 操作步骤

- 修改文件：**DevelopmentSettings.java**

- 文件路径：**alps-p25/packages/apps/Settings/src/com/android/settings/DevelopmentSettings.java**

  ```java
  +        // modify by zhanye
         /// M: CR ALPS00244115. Lock and unlock screen, the "USB debugging" is unchecked.
  -        // boolean isChecked = (mAdbDialog != null && mAdbDialog.isShowing()) ? true :
  -        //             (Settings.Global.getInt(cr, Settings.Global.ADB_ENABLED, 0) != 0);
  +        boolean isChecked = true;
  
  ```


