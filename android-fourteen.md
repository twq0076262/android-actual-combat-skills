# [Android 实战技巧之十四：混淆与反编译](http://blog.csdn.net/lincyang/article/details/44037845)

## 混淆

**Android Studio：**

只需在 build.gradle(Module:app)中的 buildTypes 中增加 release 的编译选项即可，如下：

```
    buildTypes {
           release {
               minifyEnabled true
               proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-  rules.pro'
           }
       }
```

这个 proguard-android.txt 是 sdk 中 groguard 默认的文件，具体地址在：/opt/sdk/tools/proguard/proguard-android.txt  
而 proguard-rules.pro 是 AS 中专用的 proguard 配置文件，其实只是后缀名不同，与 Eclipse 中的 proguard-project.txt 是一样的，配置规则相同，后面会详细提到。 

老版本开启混淆的命令是 runProguard,现在统一用 minifyEnabled 命令了，将其设为 true 就好了。 

编译的时候可以使用命令：

```
    ./gradlew assembleRelease
```

或者用上一篇生成签名 apk 的办法都可。

**Eclipse：** 

在 project.properties 文件中开启 proguard 配置（放开注释），如下：

```
    proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt
```

同样，生成签名时代码就会被混淆。

**混淆语法** 

请参考 ${sdk.dir}/tools/proguard/proguard-android.txt 文件，需要注意的是文件中虽然有了不混淆 Parcelable 的语句，如下：

```
    -keep class * implements android.os.Parcelable {
      public static final android.os.Parcelable$Creator *;
    }
```

但是还是要自己把继承自 Parcelable 的类写进来避免混淆，否则会出现 BadParcelableException 异常。

```
    -keep class com.linc.datatype.XXInfo {*;}
```

为微信分享而引入的 jar 包，我们不需要对其进行混淆，也需要在 proguard-android.txt 中注明，如下：

```
    -keep class com.tencent.** { *; }
    -keep class com.tencent.mm.sdk.openapi.WXMediaMessage {*;}
    -keep class com.tencent.mm.sdk.openapi.** implements com.tencent.mm.sdk.openapi.WXMediaMessage$IMediaObject {*;}
```

为了验证是否混淆成功，可以使用下面的反编译工具验证。

## 反编译

主要用到三个工具： 

dex2jar：将 dex 文件转为 jar 文件 

jd-gui：反编译 jar 文件 

AXMLPrinter2.jar：反编译 xml 文件

使用方法参见[《反编译 apk 文件，得到其源代码的方法》](http://blog.csdn.net/lincyang/article/details/6333974)

对于 Ubuntu64位，运行 jd-gui 或许会报错： 

尝试解决如下：

```
    $ sudo apt-get install libgtk2.0-0:i386 libnss3:i386 libcurl3-gnutls:i386 libidn11:i386 libpango1.0-0:i386 libpangox-1.0-0:i386 libpangoxft-1.0-0:i386  librtmp0:i386 libxft2:i386
```

又报错：

```
    $ /opt/sdk/tools/jd-gui: error while loading shared libraries: libXxf86vm.so.1: cannot open shared object file: No such file or directory
```

解决办法如下：

```
    $ sudo apt-get install libgtk2.0-0:i386 libxxf86vm1:i386 libsm6:i386 lib32stdc++6
```

---

参考： 

[http://blog.csdn.net/lincyang/article/details/6333974](http://blog.csdn.net/lincyang/article/details/6333974)