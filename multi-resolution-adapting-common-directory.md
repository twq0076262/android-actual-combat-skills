# [Android 实战技巧之十五：多分辨率适配常用目录](http://blog.csdn.net/lincyang/article/details/44174997)

一个好的 App 必须要支持绝大多数市面上的设备，适配繁多的分辨率一度让我们陷入了分辨率的海洋。无论如何，这个工作是逃不掉的。

---

我们可以用多个 layout 目录对不同分辨率进行单独布局，如下：

```
    layout-large-mdpi   (1024x600)
    layout-large-tvdpi  (800x1280)
    layout-large-xhdpi  (1200x1920)
    layout-xlarge-mdpi  (1280x800)
    layout-xlarge-xhdpi (2560x1600)
```

或者直接使用下面这样：

```
    layout-640x360
    layout-800x480
```

与 layout 对应的，有不同的 drawable：

```
    res/drawable        (default)
    res/drawable-ldpi/  (240x320 and nearer resolution)
    res/drawable-mdpi/  (320x480 and nearer resolution)
    res/drawable-hdpi/  (480x800, 540x960 and nearer resolution)
    res/drawable-xhdpi/  (720x1280 - Samsung S3, Micromax Canvas HD etc)
    res/drawable-xxhdpi/ (1080x1920 - Samsung S4, HTC one, Nexus 5, etc)
```

用不同的 layout 毕竟工作量巨大，我们的实践是用不同的 values 来对应同 layout 中的值，目录如下：

```
    res/values/dimens.xml(default)
    res/values-ldpi/dimens.xml   (240x320 and nearer resolution)
    res/values-mdpi/dimens.xml   (320x480 and nearer resolution)
    res/values-hdpi/dimens.xml   (480x800, 540x960 and nearer resolution)
    res/values-xhdpi/dimens.xml  (720x1280 - Samsung S3, Micromax Canvas HD, etc) 
    res/values-xxhdpi/dimens.xml (1080x1920 - Samsung S4, HTC one, etc)
    res/values-large/dimens.xml  (480x800)
    
    res/values-large-mdpi/dimens.xml (600x1024)
    res/values-sw600dp/dimens.xml  (600x1024)
    res/values-sw720dp/dimens.xml  (800x1280)
    res/values-xlarge-xhdpi/dimens.xml (2560x1600 - Nexus 10")
    res/values-large-xhdpi/dimens.xml  (1200x1920 - Nexus 7"(latest))
```

有时必须要考虑到密度，如下：

```
    ldpi120dpi  0.75
    mdpi160dpi  1
    hdpi240dpi  1.5
    xhdpi   320dpi  2
```