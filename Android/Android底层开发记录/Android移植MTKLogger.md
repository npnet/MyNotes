### 

## Android移植MTKLogger

平台：MTK6757

版本：Android7.0

1.MTK6757(p25)中没有MTKLogger这个apk，这次从MTK6761中将MTKLogger移植过去，只支持mobile_log的使用（其他三个没有处理）

2.步骤

- 将MTKLogger的源码复制到：**vendor/mediatek/proprietary/packages/apps/**目录中

- 修改MTKLogger的AndroidManifest.xml，使得APP不需要使用adb命令调用，直接在手机桌面可以看到

  注意修改路径不是MTKLogger根目录的AndroidManifest.xml，而是**MTKLogger/user/AndroidManifest.xml** 

  ```xml
  <activity
            android:name=".MainActivity"
            android:configChanges="keyboard|orientation|mcc|mnc"
            android:label="@string/app_name"
            android:launchMode="singleTask"
            android:screenOrientation="portrait" >
      <intent-filter>
          <action android:name="android.intent.action.MAIN" />
  - 		<category android:name="android.intent.category.DEFAULT" />
  +        <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
  </activity>
  ```

- 在**device/mediatek/mt6757/device.mk**中加入以下代码：

  ```makefile
  #################################################
  # MTKLogger
  ifeq ($(strip $(MTK_MTKLOGGER_SUPPORT)),yes)
    PRODUCT_PACKAGES += MTKLogger
    PRODUCT_PACKAGES += MTKLoggerProxy
    ifeq ($(strip $(MTK_LOG_SUPPORT_MOBILE_LOG)),yes)
      PRODUCT_PACKAGES += mobile_log_d
    endif
  endif
  ```

3.debug过程

- adb调出MTKLogger

  `adb shell am start -n com.mediatek.mtklogger/com.mediatek.mtklogger.MainActivity`

- adb启动MTKLogger(只选择mobile_log)

  `adb shell am broadcast -a com.mediatek.mtklogger.ADB_CMD -e cmd_name start --ei cmd_target 1`

4.感悟

  其实就是复制粘贴，把对应的package在device.mk里加上，这个过程主要是用了对比，对比MTK6761，用adb shell ps查看进程，用adb logcat去看对应的log