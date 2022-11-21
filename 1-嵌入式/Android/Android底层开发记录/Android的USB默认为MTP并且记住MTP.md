## Android的USB默认为MTP并且记住

平台：MTK6757

版本：Android7.0

**修改usb上盘后默认协议为MTP，并且拔出USB后不再改回Charging，继续保持MTP**

### 1. 操作步骤

路径：**frameworks/base/services/usb/java/com/android/server/usb/UsbDeviceManager.java**

```java
private final class UsbHandler extends Handler {
    ...
    ...
-   //private boolean mUsbDataUnlocked;
+	private boolean mUsbDataUnlocked = true;    // modify by zhanye
	...
	...
	@Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
                ...
                ...
                if (!mConnected) {
                     // When a disconnect occurs, relock access to sensitive user data
-                    // mUsbDataUnlocked = false;
+                    // modify by zhanye
+                    mUsbDataUnlocked = true;
                }
                updateUsbNotification();
                updateAdbNotification();
                if (UsbManager.containsFunction(mCurrentFunctions,
                    UsbManager.USB_FUNCTION_ACCESSORY)) {
                    updateCurrentAccessory();
                } else if (!mConnected) {
                    // restore defaults when USB is disconnected
-                    // setEnabledFunctions(null, false);
+                    // modify by zhanye
+                    setEnabledFunctions(null, true);
                }
```

