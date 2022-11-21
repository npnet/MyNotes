## Android编译C语言可执行文件、静态库

### 1.编译C语言可执行文件

Android中可以编译C语言，编成一个可执行的程序，然后通过adb shell 去运行(和Linux一样的操作)

首先是在development目录下创建一个文件夹，比如Hello目录。

然后在external/Hello目录中编写相关.c文件以及Android.mk文件

**核心是Android.mk**

下面是编译成可执行文件的Android.mk：

~~~makefile
LOCAL_PATH := $(call my-dir)
#include $(CLEAR_VARS)
LOCAL_SRC_FILES:= main.c
LOCAL_MODULE:= test_main
LOCAL_MODULE_TAGS := optional
#LOCAL_C_INCLUDES :=
#LOCAL_STATIC_LIBRARIES :=
#LOCAL_SHARED_LIBRARIES :=
include $(BUILD_EXECUTABLE)
~~~

> 解释：
>
> LOCAL_PATH := $(call my-dir) :  **返回Android.mk所在的当前路径**
>
> \#include $(CLEAR_VARS)：**清理除LOCAL_PATH变量以外的LOCAL_XXX**
>
> LOCAL_SRC_FILES：源文件
>
> LOCAL_MODULE：编译出来的模块名，make的时候指定的名字 比如这里是 make test_main
>
> include $(BUILD_EXECUTABLE)：指定生成可执行文件（EXECUTABLE）

最后生成的可执行程序test_main的路径是：out/target/product/generic/obj/EXECUTABLE 对应文件夹test_main_XXXX下（这里的generic是根据目标板子的名字会改变的)

### 2. 编译静态库

编译静态库的Android.mk和编译可执行程序的差不多的。

~~~makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_CLANG := true
LOCAL_MODULE := liblongsys_testdram
LOCAL_MODULE_TAGS := tests
LOCAL_SRC_FILES := \
	longsys_testdram.c 
	
include $(BUILD_STATIC_LIBRARY)
~~~

> 不同之处在于最后一句**include $(BUILD_STATIC_LIBRARY)**表示生成静态库

最后生成的静态库liblongsys_testdram.a的路径是：out/target/product/generic/obj/STATIC_LIBRARY对应文件夹test_main_XXXX下（这里的generic是根据目标板子的名字会改变的)

### 3. 编译可执行程序时链接静态库

有时候需要这样子操作，把源码编译成一个静态库，然后提供一个test给别人，那这时候通过上面的方式编译出静态库之后，再把静态库放到当前文件夹下，再去编译执行程序，很多时候是会报错的。

主要报错原因（当然前提是在Android.mk中已经加上了LOCAL_STATIC_LIBRARIES ：= xxx）：

`ninja: error: 'out/target/product/tk6757_66_n1/obj/STATIC_LIBRARIES/liblongsys_testdram_intermediates/export_includes', needed by 'out/target/product/tk6757_66_n1/obj/EXECUTABLES/longsys_testdram_test_intermediates/import_includes', missing and no known rule to make it`

这个问题是因为编译时候找静态库都是到*out/target/product/xxx/obj/STATIC_LIBRARIES/*下找的，但是我们现在库文件放在里当前文件夹下，所以解决方法就是使用：**LOCAL_PREBUILT_LIBS**

LOCAL_PREBUILT_LIBS 指定库的名字，然后编译时候会先复制一份到out对应目录下，这样就不会出错了。

Android.mk文件：

~~~makefile
LOCAL_PATH:= $(call my-dir)

# 这三句比较重要
include $(CLEAR_VARS)
LOCAL_PREBUILT_LIBS := liblongsys_testdram.a 
include $(BUILD_MULTI_PREBUILT)

test_includes := \
	$(LOCAL_PATH)/

include $(CLEAR_VARS)

LOCAL_CFLAGS := -I$(LOCAL_PATH)/

LOCAL_SRC_FILES := main.c

LOCAL_MODULE := longsys_testdram_test
LOCAL_MODULE_TAGS := tests
LOCAL_STATIC_LIBRARIES := liblongsys_testdram

LOCAL_C_INCLUDES := $(test_includes)

include $(BUILD_EXECUTABLE)
~~~



