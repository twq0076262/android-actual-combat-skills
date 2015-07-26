#[Android 实战技巧之十六：getprop 与 dumpsys 命令](http://blog.csdn.net/lincyang/article/details/44198189)

[`adb`](http://www.csdn.net/tag/adb) [`android`](http://www.csdn.net/tag/android) [`getprop`](http://www.csdn.net/tag/getprop) [`dumpsys`](http://www.csdn.net/tag/dumpsys)

<table class="table table-bordered table-striped table-condensed"> <tr> <td>目录(?)[+]</td> </tr> </table>

Android 设备连接 PC 后，我们可以通过 adb 命令完成绝大多数工作。下面借助 getprop、dumpsys 来了解一些系统相关信息。

##一、getprop

此命令的原理很简单，就是从系统的各种配置文件中读取信息。那么这些文件在你用 adb shell 进入设备内部后很容易找到，它们是：

```

    init.rc
    default.prop
    /system/build.prop

```

此时直接使用 cat 命令也是可以把这些信息显示出来的。 

下面列出比较常用的信息 

1.获得IP

```

    $ adb shell getprop dhcp.wlan0.ipaddress
    192.168.0.107

```

2.手机名称

```

    $ adb shell getprop ro.product.device
    Ulike2
    $ adb shell getprop ro.product.model
    U705T
    $ adb shell getprop ro.product.name
    oppo17_12035

```

3.serial number

```

    $ adb shell getprop ro.serialno
    0000012035ABCXXX

```

4.屏幕密度

```

    $ adb shell getprop ro.sf.lcd_density
    240

```

好了，只要使用 adb shell getprop 就可以把所有的信息都打印出来。而使用 setprop 命令就可以进行相对应的设置啦。

##二、dumpsys

Android 系统启动时会有大批的服务随之启动，那么我们就可以用 dumpsys 命令来查看每个服务的运行情况。作为一名 Android 开发者，我们至少要了解这些 Service 的存在：

```

    Currently running services:
      DMAgent
      NvRAMAgent
      SurfaceFlinger
      accessibility
      account
      activity
      alarm
      appwidget
      audio
      audioprofile
      backup
      battery
      batteryinfo
      bluetooth
      bluetooth_a2dp
      bluetooth_profile_manager
      bluetooth_socket
      clipboard
      connectivity
      content
      country_detector
      cpuinfo
      device_policy
      devicestoragemonitor
      diskstats
      drm.drmManager
      dropbox
      entropy
      gfxinfo
      hardware
      input_method
      iphonesubinfo
      isms
      location
      media.audio_flinger
      media.audio_policy
      media.camera
      media.mdp_service
      media.player
      meminfo
      memory.dumper
      mount
      mtk-agps
      mtk-epo-client
      netpolicy
      netstats
      network_management
      notification
      oppo.com.IRUtils
      package
      permission
      phone
      power
      samplingprofiler
      search
      sensorservice
      simphonebook
      statusbar
      telephony.registry
      telephony.registry2
      textservices
      throttle
      uimode
      usagestats
      usb
      vibrator
      wallpaper
      wifi
      wifip2p
      window

```

当我们需要知道设备的分辨率时，可以使用如下命令：

```

    $ adb shell dumpsys window displays
    WINDOW MANAGER DISPLAY CONTENTS (dumpsys window displays)
    Display: mDisplayId=0
    init=720x1280 320dpi cur=720x1280 app=720x1280 rng=720x670-1280x1230
    layoutNeeded=false

```

or

```

    $ adb shell dumpsys window
    ...
    Display: init=540x960 base=540x960 cur=540x960 app=540x888 raw=540x960

```

Refer to ： 

[http://blog.csdn.net/wangjia55/article/details/7446772](http://blog.csdn.net/wangjia55/article/details/7446772)
 
[http://blog.csdn.net/kevinx_xu/article/details/11846289](http://blog.csdn.net/kevinx_xu/article/details/11846289) 

[http://blog.csdn.net/z_guijin/article/details/8203028](http://blog.csdn.net/z_guijin/article/details/8203028)