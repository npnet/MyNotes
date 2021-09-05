## Android 关闭usb adb 授权对话框

平台：MTK6757

版本：Android 7.0

### 1.关闭usb adb 授权对话框

- 修改文件：**UsbDebuggingActivity.java**

- 文件路径：

  **alps-p25/frameworks/base/packages/SystemUI/src/com/android/systemui/usb/UsbDebuggingActivity.java**

- 步骤：

  第一步： 屏蔽onClick事件

  ```java
  @Override
      public void onClick(DialogInterface dialog, int which) {
  +    /*
          boolean allow = (which == AlertDialog.BUTTON_POSITIVE);
          boolean alwaysAllow = allow && mAlwaysAllow.isChecked();
          try {
              IBinder b = ServiceManager.getService(USB_SERVICE);
              IUsbManager service = IUsbManager.Stub.asInterface(b);
              if (allow) {
                  service.allowUsbDebugging(alwaysAllow, mKey);
              } else {
                  service.denyUsbDebugging();
              }
          } catch (Exception e) {
              Log.e(TAG, "Unable to notify Usb service", e);
          }
          finish();
  +    */
      }
  ```

  第二步：修改OnReceive事件，在usb接上时会处理这个事件

  思路：直接关闭对话框mActivity.finish()，然后设置为允许service.allowUsbDebugging(true, mKey);

  ```java
  @Override
          public void onReceive(Context content, Intent intent) {
              String action = intent.getAction();
              if (!UsbManager.ACTION_USB_STATE.equals(action)) {
                  return;
              }
   +           // modify by zhanye
   +           // boolean connected = intent.getBooleanExtra(UsbManager.USB_CONNECTED, false);
   +           boolean connected = false;
              if (!connected) {
                  mActivity.finish();
              }
  +            // add by zhanye
  +            try {
  +                IBinder b = ServiceManager.getService(USB_SERVICE);
  +                IUsbManager service = IUsbManager.Stub.asInterface(b);
  +            
  +               service.allowUsbDebugging(true, mKey);
  +            }
  +            catch (Exception e) {
  +                Log.e(TAG, "Unable to notify Usb service", e);
  +            }
          }
  ```
