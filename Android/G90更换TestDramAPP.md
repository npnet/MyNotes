### G90添加编译APP源文件

```shell
diff --git a/build/make/target/product/handheld_system.mk b/build/make/target/product/handheld_system.mk
index 4abb6e2..8f08531 100755
--- a/build/make/target/product/handheld_system.mk
+++ b/build/make/target/product/handheld_system.mk
@@ -73,7 +73,7 @@ PRODUCT_PACKAGES += \
     UserDictionaryProvider \
     VpnDialogs \
     vr \
-       TestDram2 \
+       TestDram \
        MyFish \
```

```shell
diff --git a/device/mediatek/mt6785/device.mk b/device/mediatek/mt6785/device.mk
index 0d43db3..1aa1a57 100755
--- a/device/mediatek/mt6785/device.mk
+++ b/device/mediatek/mt6785/device.mk
@@ -51,7 +51,7 @@ endif
 endif
 endif
 
-PRODUCT_PACKAGES += TestDram2  
+PRODUCT_PACKAGES += TestDram
 PRODUCT_PACKAGES += libtestdram
 PRODUCT_PACKAGES += libserialport
```

在路径 ./vendor/mediatek/proprietary/packages/3rd-party下，拷贝Testdram应用源码