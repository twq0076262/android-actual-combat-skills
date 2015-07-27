# [Android 实战技巧之十八：adb 取出安装在手机中的 apk](http://blog.csdn.net/lincyang/article/details/44418379)

场景： 

朋友看见你 Android 手机中的游戏或应用很好玩，也想装一个此程序，但限于网络条件不能从网上下载。那么最简单的办法就是直接从你手机中将此 apk 扣出来给他安装上。

## pm 命令

第一步，找到程序的包名 

借助 adb shell pm 命令，将安装的所有应用包名列出来：

```
    $ adb shell pm list packages
    package:android
    package:cn.wps.moffice
    package:com.android.backupconfirm
    package:com.android.bluetooth
    package:com.android.browser
    package:com.android.calculator2
    package:com.android.camera
    package:com.android.certinstaller
    package:com.android.contacts
```

第二步，找到 apk 的位置

```
    $ adb shell pm path com.tence01.mm
    package:/data/app/com.tence01.mm-1.apk
```

第三步，pull 出来

```
    $ adb pull /data/app/com.tence01.mm-1.apk ~/apks
    2407 KB/s (25567735 bytes in 10.370s)
```

## root 的手机会更好办

```
    $ adb shell
    shell@android:/ $ su
    shell@android:/ # cd data/app
    shell@android:/data/app # ls
    com.android.update.dmp-2.apk
    com.baidu.superservice-1.apk
    com.tence01.mm-1.apk
    com.tencent.mm-1.apk
```

或者直接搜索你要的 apk：

```
    shell@android:/ # find -name *.apk
    ./udisk/我的下载/download/我的应用/aqgj_1365562277812.apk
```