# [Android实战技巧之二十三：Android Studio的NDK开发](http://blog.csdn.net/lincyang/article/details/44725529)

<table class="table table-bordered table-striped table-condensed"> <tr> <td>目录(?)[+]</td> </tr> </table>

较新的 NDK 版本是 r10b，Android Studio 对 NDK 开发的支持还处于构思阶段，所以很多工作比如用 Javah 生成头文件等工作还要自己做。今天用一个例子来演示 AS 中的 NDK 开发。

## 新建一个项目 SecondNdkTest

在此项目中新建一个 Module 叫 ndklibrary，作为 so 库单独划出来。在 library 中新建一个 Java 类 SecondLib，内容如下：

```
    package com.linc.ndklibrary;

    /**
     * Created by linc on 15-3-29.
     */
    public class SecondLib {
        // Native implementation  
        static {  
        System.loadLibrary("SecondLib"); 
        } 
        //int array
        public static native int[] intMethod();
        //string array
        public static native String[] stringMethod();
    }
```

Build—>Make Module’ndklibrary’,这样 SecondLib 就编译完成了，通过 SecondLib.class，用 Javah 生成 C 的头文件，如下：

```
    AndroidStudioProjects/SecondNdkTest/ndklibrary/src/main$ javah -d jni -classpath ../../build/intermediates/classes/debug com.linc.ndklibrary.SecondLib
    AndroidStudioProjects/SecondNdkTest/ndklibrary/src/main$ ls
    AndroidManifest.xml  java  jni  res
    AndroidStudioProjects/SecondNdkTest/ndklibrary/src/main$ ls jni/
    com_linc_ndklibrary_SecondLib.h
```

头文件内容如下：

```
    /* DO NOT EDIT THIS FILE - it is machine generated */
    #include <jni.h>
    /* Header for class com_linc_ndklibrary_SecondLib */
    
    #ifndef _Included_com_linc_ndklibrary_SecondLib
    #define _Included_com_linc_ndklibrary_SecondLib
    #ifdef __cplusplus
    extern "C" {
    #endif
    /*
     * Class: com_linc_ndklibrary_SecondLib
     * Method:intMethod
     * Signature: ()[I
     */
    JNIEXPORT jintArray JNICALL Java_com_linc_ndklibrary_SecondLib_intMethod
      (JNIEnv *, jclass);
    
    /*
     * Class: com_linc_ndklibrary_SecondLib
     * Method:stringMethod
     * Signature: ()[Ljava/lang/String;
     */
    JNIEXPORT jobjectArray JNICALL Java_com_linc_ndklibrary_SecondLib_stringMethod
      (JNIEnv *, jclass);
    
    #ifdef __cplusplus
    }
    #endif
    #endif
```

## Native 代码实现

在 jni 目录中新建 c 文件 SecondLib.c 与头文件对应，分别实现上述两个方法，内容如下：

```
    #include "com_linc_ndklibrary_SecondLib.h"
  
     const static int length=10;

     //int array
     JNIEXPORT jintArray JNICALL Java_com_linc_ndklibrary_SecondLib_intMethod
       (JNIEnv *env, jclass obj)
     {
        jintArray array;
        array=(*env)->NewIntArray(env,10);
        int i=1;
        for(;i<=10;++i)
        {
            (*env)->SetIntArrayRegion(env,array,i-1,1,&i);
        }

        //get array length
        int len=(*env)->GetArrayLength(env,array);
  
        //array content
        jint* elems=(*env)->GetIntArrayElements(env,array,0);
 
        return array;
     } 

     //string array
     JNIEXPORT jobjectArray JNICALL Java_com_linc_ndklibrary_SecondLib_stringMethod
       (JNIEnv *env, jclass obj)
     {
        jclass class=(*env)->FindClass(env,"java/lang/String");
        jobjectArray string=(*env)->NewObjectArray(env,(jsize)  length,
                class,0);
        jstring jstr;
        char* _char[]={"my ","name ","is ",
                    "linc!!","正在","学习",
                        "JNI","和","NDK","技术！"
                        };
        int i=0;
        for(;i<length;++i)
        {
            jstr=(*env)->NewStringUTF(env,_char[i]);
            (*env)->SetObjectArrayElement(env,string,i,jstr);
        }

         return string;
    }
```

## 编译

在 local.properties 中加入 ndk路径：

```
    ndk.dir=/opt/ndk/android-ndk-r10b
```

然后在 ndklibrary 的 build.gradle 中 defaultConfig 中加入 ndk 定义，如下：

```
    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
        ndk{
            moduleName "SecondLib"
        }
    }
```

这样就可以直接编译了，不用自己编写 make 文件了。 

Build—>Make Module’ndklibrary’,生成的 so 如下：

```
    AndroidStudioProjects/SecondNdkTest$ find -name *.so
    ./ndklibrary/build/intermediates/ndk/debug/lib/armeabi/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/lib/armeabi-v7a/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/lib/mips/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/lib/x86/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/obj/local/armeabi/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/obj/local/armeabi-v7a/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/obj/local/mips/libSecondLib.so
    ./ndklibrary/build/intermediates/ndk/debug/obj/local/x86/libSecondLib.so
```

注： 

至于直接在 Activity 中用 native 的方法请参考下面的前两个链接。我遇到了问题没有得到解决：

```
    $ javah -d jni -classpath /opt/sdk/platforms/android-5.1/android.jar;../../build/intermediates/classes/debug   com.linc.secondndktest.MainActivity
    Error: no classes specified
    bash: ../../build/intermediates/classes/debug/: Is a directory
```

参考：

[http://blog.csdn.net/rznice/article/details/42295215](http://blog.csdn.net/rznice/article/details/42295215) 

[http://blog.csdn.net/sodino/article/details/41946607](http://blog.csdn.net/sodino/article/details/41946607) 

[http://stackoverflow.com/questions/10483959/javah-error-android-app-activity-not-found](http://stackoverflow.com/questions/10483959/javah-error-android-app-activity-not-found) 

[http://blog.csdn.net/lincyang/article/details/6705143](http://blog.csdn.net/lincyang/article/details/6705143)