#[Android 实战技巧之三十五：了解 native activity](http://blog.csdn.net/lincyang/article/details/46474319)

[`ndk`](http://www.csdn.net/tag/ndk) [`native-act`](http://www.csdn.net/tag/native-act) [`activity`](http://www.csdn.net/tag/activity)

<table class="table table-bordered table-striped table-condensed"> <tr> <td>目录(?)[+]</td> </tr> </table>

##1.native activity 的意义

很多人觉得 Android 的 Fwk 提供的支持足够好了，既然 Google 不推荐用 Ndk 开发为什么又放宽 Ndk 的限制而推出可以无 Java 开发 Android App 呢？我的理解是不同的技术实现会有其适合的场景。 

Ndk 的适用场景官方给出三点：1.平台间的 App 移植 2.复用现有库 3.对软件性能要求较高的场合比如游戏等。那么 native activity 在十分适合游戏领域，比如 cocos-2dx 对其的使用。

##2.初步了解 native activity

借助 SDK 提供的 NativeActivity 类，我们可以创建完全的本地 activity 而无须编写 Java 代码。
 
需要注意的是，即使是无 Java 代码编写的应用仍然是跑（运行）在自己的虚拟机中以此与其他应用隔离无不影响。你可以通过 JNI 的方式调用 Fwk 层的 API，有些像 sensor、输入事件等操作可以直接调用本地接口（native interfaces）来完成（linc 注：这样才高效嘛）。

有两种方式可以实现 native activity。 

1）native_activity.h 

2)android_native_app_glue 

由于第二种方法启用另一个线程处理回调和输入事件，Ndk 的例子中就采用了这个实现方式。

##3.Ndk 自带的例子

这个程序主要演示根据 sensor 的检测结果在整个屏幕上绘制不同的颜色。
 
**AndroidManifest.xml** 

为了正常使用 native activity，我们需要把 API 级别定在 9 及以上。

```

    <uses-sdk android:minSdkVersion="9" />

```

由于我们只使用 native code，将 android:hasCode 设为 false。

```

    <application android:label="@string/app_name" android:hasCode="false">

```

声明 NativeActivity 类

```

        <activity android:name="android.app.NativeActivity"
                android:label="@string/app_name"
                android:configChanges="orientation|keyboardHidden">
            <!-- Tell NativeActivity the name of or .so -->
            <meta-data android:name="android.app.lib_name"
                    android:value="native-activity" />
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

```

注意 meta-data 中的 lib_name（native-activity）要与 Android.mk 中 LOCAL_MODULE 相同。

**Android.mk**
 
强调模块名称和源文件如下：

```

    LOCAL_MODULE    := native-activity
    LOCAL_SRC_FILES := main.c

```

**外部库的依赖** 

例子中用到了如下几个外部库，log、android（为 NDK 提供的标准 Android 支持 API）、EGL（图形 API）以及 OpenGL ES（android 用的 OpenGL，依赖 EGL） 

书写约定为上述库前缀为 -l，如下：

```

    LOCAL_LDLIBS    := -llog -landroid -lEGL -lGLESv1_CM

```

注意：
 
实际的库文件名约定为前缀为 lib，后缀为.so。比如 log 库文件名实际为 liblog.so。 

这些库的实际位置：

```

    <ndk>/platforms/android-<sdk_version>/arch-<abi>/usr/lib /

```

比如 liblog.so：

```

    $ locate liblog.so
    /opt/android-ndk-r10b/platforms/android-12/arch-arm/usr/lib/liblog.so
    /opt/android-ndk-r10b/platforms/android-12/arch-mips/usr/lib/liblog.so
    /opt/android-ndk-r10b/platforms/android-12/arch-x86/usr/lib/liblog.so
    /opt/android-ndk-r10b/platforms/android-13/arch-arm/usr/lib/liblog.so
    ...

```

**静态库** 

本例用 android_native_app_glue 管理 NativeActivity 生命周期事件：

```

    LOCAL_STATIC_LIBRARIES := android_native_app_glue

```

我们需要告知编译系统去 build 这个 static library，加上如下语句：

```

    $(call import-module,android/native_app_glue)

```

**源代码**
 
主要源代码文件只有一个，main.c。 

引入的头文件对应在 Android.mk 中提到的，如下：

```

    #include <EGL/egl.h>
    #include <GLES/gl.h>
    
    #include <android/sensor.h>
    #include <android/log.h>
    #include <android_native_app_glue>

```

程序的入口是 android_main，通过 android_native_app_glue 调入并传入一个预定义 state 结构来管理 NativeActivity 的回调。

```

    void android_main(struct android_app* state)

```

结构 android_app 的定义参见/sources/android/native_app_glue/android_native_app_glue.h

接下来程序通过 glue 库来处理事件队列，参考如下代码：

```

    struct engine engine;

    // Make sure glue isn't stripped.
    app_dummy();

    memset(&engine, 0, sizeof(engine));
    state->userData = &engine;
    state->onAppCmd = engine_handle_cmd;
    state->onInputEvent = engine_handle_input;
    engine.app = state;

```

**准备sensor**

```

    // Prepare to monitor accelerometer
    engine.sensorManager = ASensorManager_getInstance();
    engine.accelerometerSensor = ASensorManager_getDefaultSensor(engine.sensorManager,
            ASENSOR_TYPE_ACCELEROMETER);
    engine.sensorEventQueue = ASensorManager_createEventQueue(engine.sensorManager,
            state->looper, LOOPER_ID_USER, NULL, NULL);

```

**处理消息循环**

```

    while (1) {
            // Read all pending events.
            int ident;
            int events;
            struct android_poll_source* source;


            // If not animating, we will block forever waiting for events.
            // If animating, we loop until all events are read, then continue
            // to draw the next frame of animation.
            while ((ident=ALooper_pollAll(engine.animating ? 0 : -1, NULL,
    &events,
                    (void**)&source)) >= 0) {

 
                // Process this event.
                if (source != NULL) {
                    source->process(state, source);
                }


                // If a sensor has data, process it now.
                if (ident == LOOPER_ID_USER) {
                    if (engine.accelerometerSensor != NULL) {
                        ASensorEvent event;
                        while (ASensorEventQueue_getEvents(engine.sensorEventQueue,
                                &event, 1) > 0) {
                            LOGI("accelerometer: x=%f y=%f z=%f",
                                    event.acceleration.x, event.acceleration.y,
                                    event.acceleration.z);
                        }
                    }
                }


            // Check if we are exiting.
            if (state->destroyRequested != 0) {
                engine_term_display(&engine);
                return;
            }
        }

```

**队列为空时，调用 OpenGL 绘制屏幕**

```

        if (engine.animating) {
            // Done with events; draw next animation frame.
            engine.state.angle += .01f;
            if (engine.state.angle > 1) {
                engine.state.angle = 0;
            }

            // Drawing is throttled to the screen update rate, so there
            // is no need to do timing here.
            engine_draw_frame(&engine);
        }

```

**编译**
 
首先通过 NDK 编译出 so 文件，在根目录下直接执行 ndk-build 即可：

```

    $ ndk-build
    [armeabi-v7a] Compile thumb  : native-activity <= main.c
    [armeabi-v7a] Compile thumb  : android_native_app_glue <= android_native_app_glue.c
    [armeabi-v7a] StaticLibrary  : libandroid_native_app_glue.a
    [armeabi-v7a] SharedLibrary  : libnative-activity.so
    [armeabi-v7a] Install: libnative-activity.so => libs/armeabi-v7a/libnative-activity.so

```

上述只摘录出 armeabi-v7a 一个平台的编译 log，由于在 Application.mk 中没有指明特定平台，编译系统会编译出其他 armeabi、x86 和 mips 平台的 so。

然后再编译 apk 

我尝试将工程向 AS 中导入，发现没能配置好 gradle 编译环境，无法编译。 

接着我就借助 ant 来编译，参考如下步骤：

```

    $ android update project -p .
    Updated and renamed default.properties to project.properties
    Updated local.properties
    No project name specified, using Activity name 'NativeActivity'.
    If you wish to change it, edit the first line of build.xml.
    Added file ./build.xml
    Added file ./proguard-project.txt
    
    $ ant debug
    ...
    [echo] Debug Package: /opt/android-ndk-r10b/samples/native-activity/bin/NativeActivity-debug.apk
    ...
    BUILD SUCCESSFUL
    Total time: 3 seconds

```

**最后**
 
安装运行吧！

```

    adb install /opt/android-ndk-r10b/samples/native-activity/bin/NativeActivity-debug.apk

```

参考：
 
[http://blog.csdn.net/lincyang/article/details/40950153](http://blog.csdn.net/lincyang/article/details/40950153)
