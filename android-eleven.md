# [Android 实战技巧之十一：Android Studio 和 Gradle](http://blog.csdn.net/lincyang/article/details/43853259)

经过两个多月的 AS 体验，我认为是时候将 Android 的开发环境迁移到 AS 上了。目前最新版本是 1.0.2(大年 30 当天升级到 1.1.0)，除了 UI 控件拖拽偶尔崩溃的问题（Ubuntu），其他功能用来还是十分流畅和高效。打动我的有如下几个特色：

智能感知体验特好，堪比 VS

- 布局预览，手写布局后预览页面即时显示，便于布局调整和优化

- 编辑速度飞快流畅，毫无 eclipse 的卡顿

- 布局或源码中有图标和颜色的预览，十分直观

- 调试时体验极佳

- 集成了Terminal，喜欢命令行操作的伙伴不用额外启动终端了。

总之一句话，就是用起来特别爽！

Android Studio 源于 [IntelliJ IDEA](http://blog.csdn.net/lincyang/article/details/www.jetbrains.com/idea/) 的社区版，构建工具是 [Gradle](http://www.gradle.org/) 这个下一代的构建工具,再加上 Google 为 Android 定制的一些工具，那么 AS 必然会成为 Android 开发工具的经典款。

## Android Studio 的安装

Adnroid 官网上不去，我们可以到其他[网站下载](http://www.android-studio.org/index.php/88-download/) AS，然后再升级到 1.0.2。
 
AS 对系统的要求不低，不过我这 i7 处理器 +8G 内存还是毫无压力啊。

Windows

`Microsoft® Windows® 8/7/Vista/2003 (32 or 64-bit)`
`2 GB RAM minimum, 4 GB RAM recommended`
`400 MB hard disk space`
`At least 1 GB for Android SDK, emulator system images, and caches`
`1280 x 800 minimum screen resolution`
`Java Development Kit (JDK) 7`
`Optional for accelerated emulator: Intel® processor with support for Intel® VT-x, Intel® EM64T (Intel® 64), and Execute Disable (XD) Bit functionality`

Mac OS X

`Mac® OS X® 10.8.5 or higher, up to 10.9 (Mavericks)`
`2 GB RAM minimum, 4 GB RAM recommended`
`400 MB hard disk space`
`At least 1 GB for Android SDK, emulator system images, and caches`
`1280 x 800 minimum screen resolution`
`Java Runtime Environment (JRE) 6`
`Java Development Kit (JDK) 7`
`Optional for accelerated emulator: Intel® processor with support for Intel® VT-x, Intel® EM64T (Intel® 64), and Execute Disable (XD) Bit functionality`

On Mac OS, run Android Studio with Java Runtime Environment (JRE) 6 for optimized font rendering. You can then configure your project to use Java Development Kit (JDK) 6 or JDK 7. 

Linux

`GNOME or KDE desktop`
`GNU C Library (glibc) 2.11 or later`
`2 GB RAM minimum, 4 GB RAM recommended`
`400 MB hard disk space`
`At least 1 GB for Android SDK, emulator system images, and caches`
`1280 x 800 minimum screen resolution`
`Oracle® Java Development Kit (JDK) 7`

下载后将其解压到你指定的路径。我在 Ubuntu 下工作，就直接将其放到/opt 下了。解压后内容如下：

```
    android-studio3$ ls
    bin  build.txt  gradle  Install-Linux-tar.txt  lib  license  LICENSE.txt  NOTICE.txt  plugins
```

值得一说的是，gradle 就在这里，一会儿我们可以直接用 gradle 去做简单的编译工作。首先，我们要执行 bin 下的 studio.sh 启动 AS，就像 Install-Linux-tar.txt 中说的，我们可以将这个 bin 目录放到系统变量中，以后启动 AS 只需输入 studio.sh 即可。比如我在 .bashrc 中添加如下内容：

```
    export PATH="$PATH:/opt/android-studio3/bin"
    export PATH="$PATH:/opt/android-studio3/gradle/gradle-2.2.1/bin"
```

首次启动会检测 sdk 并升级到最新，如果不用代理，这一步我们无法通过，AS 就不会启动成功。解决办法就是将自己的 Adnroid SDK Manager 配好代理到国内的镜像，请参照[《Android 实战技巧之九：最新 Android 开发环境（Eclipse+ADT+Android 5.0）》](http://blog.csdn.net/lincyang/article/details/42029257) ，顺利通过升级后，AS 会成功启动。后面的事情就简单了，界面清晰明了，就像你用其他 IDE 一样，上手很快。但是项目结果变化很大（与 Eclipse 相比），快捷键变化也很大，都要适应一段时间。下载一份 [Keymap](https://www.jetbrains.com/idea/docs/IntelliJIDEA_ReferenceCard.pdf) 打印出来，用到了就看看，会很快进入状态。

Tips： 

打开项目后修改 sdk 和 jdk 路径，设置如下：File –>Other Settings –>Default project Structure 

如果你喜欢黑色风格的主题，那么切换到吸血鬼 Darcula 主题是个不错的选择：File–>Settings–>Appearance–>Theme

## Gradle

项目中有两个 build.gradle 文件，如下：

```
    $ find -name build.gradle
    ./app/build.gradle
    ./build.gradle
```

项目根目录下的 build.gradle 只做了比较 commen 的配置，app 下的 build.gradle 是针对此 app 更细致的配置：

```
    apply plugin: 'com.android.application'

    android {
        compileSdkVersion 21
        buildToolsVersion "21.1.2"

        defaultConfig {
            applicationId "com.linc.arrowfall"
            minSdkVersion 17
            targetSdkVersion 21
            versionCode 1
            versionName "1.0"
        }
        buildTypes {
            release {
                minifyEnabled false
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
        }
    }
 
    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile 'com.android.support:appcompat-v7:21.0.3'
    }
```

Make Project（Ctrl+F9）时，右下角 Gradle Console 就会打印如下信息：

```
    Executing tasks: [:app:compileDebugSources]

    Configuration on demand is an incubating feature.
    :app:preBuild
    :app:preDebugBuild
    :app:checkDebugManifest
    :app:preReleaseBuild
    :app:prepareComAndroidSupportAppcompatV72103Library UP-TO-DATE
    :app:prepareComAndroidSupportSupportV42103Library UP-TO-DATE
    :app:prepareDebugDependencies
    :app:compileDebugAidl UP-TO-DATE
    :app:compileDebugRenderscript UP-TO-DATE
    :app:generateDebugBuildConfig UP-TO-DATE
    :app:generateDebugAssets UP-TO-DATE
    :app:mergeDebugAssets UP-TO-DATE
    :app:generateDebugResValues UP-TO-DATE
    :app:generateDebugResources UP-TO-DATE
    :app:mergeDebugResources UP-TO-DATE
    :app:processDebugManifest UP-TO-DATE
    :app:processDebugResources UP-TO-DATE
    :app:generateDebugSources UP-TO-DATE
    :app:compileDebugJava
    :app:compileDebugNdk
    :app:compileDebugSources

    BUILD SUCCESSFUL

    Total time: 10.23 secs
```

先放下 AS 中的 Gradle，我们先从 Gradle 命令行说起。刚刚提到 AS 中自带的 Gradle 路径在 android-studio3/gradle/gradle-2.2.1/bin 下，将其加入到环境变量（如上），这样在如何位置都可以使用 gradle 工具了。下面来作一下 gradle 最简单的使用：

    $ gradle -v
    
    ------------------------------------------------------------
    Gradle 2.2.1
    ------------------------------------------------------------

    Build time:   2014-11-24 09:45:35 UTC
    Build number: none
    Revision: 6fcb59c06f43a4e6b1bcb401f7686a8601a1fb4a
    
    Groovy:   2.3.6
    Ant:  Apache Ant(TM) version 1.9.3 compiled on December 23 2013
    JVM:  1.7.0_71 (Oracle Corporation 24.71-b01)
    OS:   Linux 3.13.0-45-generic amd64

gradle 是正常工作了，下面来个 hello world 吧。新建一个 build.gradle 文件，加入如下代码：

```
    task helloworld << {
            println 'hello world'
    }
```

这个 task 只输出一条 log，执行如下命令：

```
    $ gradle -q helloworld
    hello world
```

参数 -q 只是打印 log，这个 task 也就是此功能而已。 
在此目录下执行 gradle –gui,调出图形界面的 gradle，看看 helloworld 的其他信息。

**编译 Java 程序**

现在尝试编译一个最简单的 Java 程序，在刚刚的目录下新建目录和文件如下：

```
    $ mkdir -p src/main/java/com/linc; vim src/main/java/com/linc/HelloWorld.java
```

代码内如如下：

```
    package com.linc;

    public class HelloWorld {
        public static void main(String args[]) {
            System.out.println("hello, world");
        }
    }
```

build.gradle 文件与 src 目录平级，内如只有一行：

```
    apply plugin: 'java'
```

此时运行 gradle build:

```
    $ gradle build
    :compileJava
    :processResources UP-TO-DATE
    :classes
    :jar
    :assemble
    :compileTestJava UP-TO-DATE
    :processTestResources UP-TO-DATE
    :testClasses UP-TO-DATE
    :test UP-TO-DATE
    :check UP-TO-DATE
    :build
    
    BUILD SUCCESSFUL
    
    Total time: 2.206 secs
```

此时的目录结构变为如下所示：

```
    $ tree -L 6
    .
    ├── build
    │   ├── classes
    │   │   └── main
    │   │       └── com
    │   │           └── linc
    │   │               └── HelloWorld.class
    │   ├── dependency-cache
    │   ├── libs
    │   │   └── helloworld.jar
    │   └── tmp
    │       ├── compileJava
    │       └── jar
    │           └── MANIFEST.MF
    ├── HelloWorld.java
    └── src
        └── main
            └── java
                └── com
                    └── linc
                        └── HelloWorld.java
```

运行编译好的 Java 程序:

```
    $ java -cp build/classes/main/ com.linc.HelloWorld
    hello, world
```

Gradle 的初体验就到这里，更复杂的构建任务还在后头。有了 AS 这个强大的工具，Android 开发会变得越来越有乐趣！

参考：

[http://www.gradle.org/documentation](http://www.gradle.org/documentation)

[http://www.android-studio.org/index.php/88-download/](http://www.android-studio.org/index.php/88-download/)