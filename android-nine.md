#[Android 实战技巧之九：最新 Android 开发环境（Eclipse+ADT+Android 5.0）](http://blog.csdn.net/lincyang/article/details/42029257)

##一、一切由运行时错误引起

dalvikvm Could not find class '引用包.类', referenced from method... 

其实在编译时也会见到如下错误：

   [dx] 

   [dx] trouble processing:

   [dx] bad class file magic (cafebabe) or version (0033.0000)

   [dx] ...while parsing com/novell/sasl/client/DigestChallenge.class

   [dx] ...while processing com/novell/sasl/client/DigestChallenge.class

##二、尝试

###1.使用 JDK7

有推荐使用 JDK7来解决类似问题的帖子，可是我照做并没有解决问题。

###2.升级 build-tools

先来说说我的开发环境吧。

那是在2013年末配置的环境，ADT 大版本号是22，tools 大版本号也是22，Plantform-tools 已经是最新的21，build-tools 是18.1.1。

经过同事的实践，问题应该归咎于 build-tools 版本低的缘故。

##三、最新开发环境的搭建（Eclipse & ADT&SDK）
 
###1.各 tools 的升级

这里我们要重新配置一下代理，去 neusoft.edu.cn 镜像网站中下载最新的工具和 SDK。

启动 Android SDK Manager（命令行中直接输入android），Tools--->Options...，弹出 Android SDK Manager - Settings 窗口；在HTTP Proxy Server 和 HTTP Proxy Port 输入框内填入 `mirrors.neusoft.edu.cn`(注意没有http等前缀)和80，并且选中Force https://... sources to be fetched using http://...复选框。 再选择 Packages--->Reload。

此时会发现我们顺利的取到 Packages 了，那么我们尽情下载吧。除了最新的 Android 5.0.1 还没有提供，其他的一应俱全了。

tools 更新到最新是24.0.2，
build-tools 我选择了19.1、20 和21.1.2，分别对应 API19（4.4.2）、API20（L）和 API21（5.0）.

Android 5.0 全部选择。

###2.ADT 的升级

由于更 tools 升级到最新，那么 ADT22已经过期了，需要使用23及以上版本的 ADT。找到好心人上传的23.03，安装时发现 eclipse 版本不支持最新的 ADT（我使用的Juno），好吧，既然这样就都来新的吧。

###3.eclipse luna

最新的版本是 luna，还是130多兆。解压后直接启动。

###4.再次安装 ADT

这时在 Help--->Install New Software --->Add, 选择 ADT23.03 ZIP 包，将“Contact all update sites during install to find required software.”勾选掉。
继续完成安装。

一切准备就绪，将之前有问题的项目引入进来，编译，出现内存方面的问题。

##四、dex 的问题

出现了两个问题：

1.unable to execute dex:java heap space

2.Conversion to Dalvik format failed: Unable to execute dex: GC overhead limit exceeded 

配置 eclipse.ini，将 Xms40m 和 Xmx512m 修改成126m 和1024m，这个值要根据自己机器配置调整，只要运行良好就 ok。

```
    $ cat eclipse.ini   
    -startup  
    plugins/org.eclipse.equinox.launcher_1.3.0.v20140415-2008.jar  
    --launcher.library  
    plugins/org.eclipse.equinox.launcher.gtk.linux.x86_64_1.1.200.v20140603-1326  
    -product  
    org.eclipse.epp.package.java.product  
    --launcher.defaultAction  
    openFile  
    -showsplash  
    org.eclipse.platform  
    --launcher.XXMaxPermSize  
    256m  
    --launcher.defaultAction  
    openFile  
    --launcher.appendVmargs  
    -vmargs  
    -Dosgi.requiredJavaVersion=1.6  
    -XX:MaxPermSize=256m  
    -Xms126m  
    -Xmx1028m  
```

重启、clean 项目，编译，通过！运行，正常！至此我的最新 Android 开发环境搭建完成。

##五、结论

时刻保持与时俱进的心态，稳定的新工具对我们的工作益处多多。

