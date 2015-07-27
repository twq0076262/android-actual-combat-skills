# [Android 实战技巧之二十七：Maven 编译开源二维码扫描项目 zxing](http://blog.csdn.net/lincyang/article/details/45287843)

拥有自己的手机软件工具箱是件非常有意义的事情。就目前国内 Android 的生态环境来说，混乱的不能再乱了。由于我们登录不了官网 App 商店，下软件就只好在国内五花八门的软件市场下载。由于这些市场的监管不力，什么样的软件都有，就拿二维码扫描软件来说，好多都带那种狗皮膏药一样的广告插件，真是特别讨厌。
 
在开源世界中有很多优秀的软件，其中 zxing 就是非常好的 Android 扫碼工具软件。我们可以拿来即用还可以学习内部机制，然后做些定制化个性化。既可以自己享用，又可以跟大家分享。真是不错。

zxing 在 github：[https://github.com/zxing/zxing](https://github.com/zxing/zxing)

```
    zxing-master$ ls
    android       android-integration  AUTHORS  CONTRIBUTING.md  core   javase  pom.xml    src                zxingorg
    android-core  androidtest          CHANGES  COPYING          glass  NOTICE  README.md  zxing.appspot.com
```

源码很多，里面的 pom.xml 告诉我们需要用 maven 编译。可惜这个构建工具我用的并熟练，一切都要摸索着来。

Maven 官网：[https://maven.apache.org](https://maven.apache.org/) 

在ubuntu下的安装是很简单的，下载 [apache-maven-3.3.1-bin.zip](http://mirror.bit.edu.cn/apache/maven/maven-3/3.3.1/binaries/apache-maven-3.3.1-bin.zip) 解压（unzip）到你喜欢的目录下如 /opt/apache-maven-3.3.1/ 。并将环境变量设置好，~/.bashrc 下填入下面内容：

```
    #Maven 
    export PATH="$PATH:/opt/apache-maven-3.3.1/bin"
    export MAVEN_OPTS="-Xms256m -Xmx512m"
```

前提是你的 Java7环境已经配好。请参考 [Android 实战技巧之八：Ubuntu 下切换 JDK 版本](http://blog.csdn.net/lincyang/article/details/42024565)

下面是我的 mvn 环境：

```
    $ mvn -v
    Apache Maven 3.3.1 (cab6659f9874fa96462afef40fcf6bc033d58c1c; 2015-03-14T04:10:27+08:00)
    Maven home: /opt/apache-maven-3.3.1
    Java version: 1.7.0_71, vendor: Oracle Corporation
    Java home: /opt/jdk/jdk1.7.0_71/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "linux", version: "3.13.0-49-generic", arch: "amd64", family: "unix"
```

对于 zxing 的编译，我没有找到相关的文档，所以都是我这个 maven 新人不断的摸索，虽然不是特别正规，但是达到我的目的是真的。

先在 zxing 根目录下执行 mvn compile，好家伙，一个劲的下载依赖包，都说这第一次编译需要下载大量的包，如果我没有做代理或库的更改，那么需要漫长的等待。

我晚上跑步将近一个小时，以为回来就编译好了。可是令我大失所望，虽然包下完了，但是编译有问题。再次执行，这次编译没有报错，但是我搜遍目录没有找到 jar 包。这是有问题的，我还是按照自己的节奏来工作吧。

android 目录是一个 eclipse 项目，我直接转换为 AS 工程然后编译发现少了好多 zxing 的类。

android-core 下的 pom 是这样的：

```
      <artifactId>android-core</artifactId>
      <version>3.2.1-SNAPSHOT</version>
      <packaging>jar</packaging>
```

没有其他依赖，直接编译成 jar。我执行 mvn package，漫长的等待后 jar 包编译出来了。

```
    $ ls android-core/target/
    android-core-3.2.1-SNAPSHOT.jar
```

我引入这个 jar，发现里面只有一个类
com.google.zxing.client.android.camera.CameraConfigurationUtils  

这显然还不够。 

core 目录才是重点，同样 mvn package 再等待，如果中途遇到依赖其他目录的 jar 就去编译之。

```
    $ ls core/target/
    core-3.2.1-SNAPSHOT.jar
```

再将其引入 android 工程，编译成功！