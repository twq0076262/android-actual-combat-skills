# [Android 实战技巧之十：获得屏幕物理尺寸、密度及分辨率](http://blog.csdn.net/lincyang/article/details/42679589)

<table class="table table-bordered table-striped table-condensed"> <tr> <td>目录(?)[+]</td> </tr> </table>

<I>大家帮忙喽！

博主参加2014博客之星活动，大家帮忙投票啦！[猛击这里](http://vote.blog.csdn.net/blogstar2014/selection?username=lincyang#content)！</I>

通过程序去了解硬件情况是一件十分有意思的事情。很早我就研究在 WM6.5上获得屏幕物理尺寸，但一直没有成功。后来又想要在 Android 上有所突破，不过在今天之前得到的尺寸都不准确。虽然很多人认为没必要这么较真，因为貌似很多情况下用不到。不过我就当这是一件很有挑战性的事，一定要做到。对，就是这么任性。

源码中 android.view 包下的 Display 类提供了很多方法供程序员获得显示相关的信息，通过此类让我们开启了解设备屏幕之旅吧。

## 一、分辨率

需要注意的原来经常使用的 getHeight() 与 getWidth() 已经不推荐使用了，建议使用 getSize()来替代。

此方法原型如下：

```
    public void getSize(Point outSize) {  
        synchronized (this) {  
            updateDisplayInfoLocked();  
            mDisplayInfo.getAppMetrics(mTempMetrics, mDisplayAdjustments);  
            outSize.x = mTempMetrics.widthPixels;  
            outSize.y = mTempMetrics.heightPixels;  
        }  
    }  
```

参数是一个返回参数，用以返回分辨率的 Point，这个 Point 也比较简单，我们只需要关注 x 和 y 这两个成员就可以了。

用法如下：

```
    private void getDisplayInfomation() {  
        Point point = new Point();  
        getWindowManager().getDefaultDisplay().getSize (point);  
        Log.d(TAG,"the screen size is "+point.toString());  
    }  
```

结果如下：

```
    D/MainActivity﹕ the screen size is Point(800, 1280)  
```

此外 Display 又提供了一个 getRealSize 方法，原型如下：

```
    public void getRealSize(Point outSize) {  
        synchronized (this) {  
            updateDisplayInfoLocked();  
            outSize.x = mDisplayInfo.logicalWidth;  
            outSize.y = mDisplayInfo.logicalHeight;  
        }  
    }  
```

从两个方法的实现上看是有区别的，但是在通常情况下二者的返回值相同。那么差异究竟在哪里，下面做一些实验来验证一下。

首先，我将 Acitvity 设置不同的 theme，比如：

```
    android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen"  
    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"  
```

结果还是相同的。

接下来将我的 Activity 父类变成 ActionBarActivity,如下：
public class MainActivity extends ActionBarActivity 
期望 ActionBar 会占用一些屏幕，并在程序中动态设置 Listview的Item 中的图片大小。在机缘巧合之下，结果验证了在这种情况下，getSize 返回的结果变了。

代码如下：

```
    private void getDisplayInfomation() {  
        Point point = new Point();  
        getWindowManager().getDefaultDisplay().getSize(point);  
        Log.d(TAG,"the screen size is "+point.toString());  
        getWindowManager().getDefaultDisplay().getRealSize(point);  
        Log.d(TAG,"the screen real size is "+point.toString());  
    }  
```

Log 如下：

```
    D/MainActivity﹕ the screen size is Point(800, 1202)  
    D/MainActivity﹕ the screen real size is Point(800, 1280)  
```

如果你不能够轻易复现也不用急，保险起见，为了得到相对正确的信息还是使用 getRealSize() 吧。

## 二、屏幕尺寸

设备的物理屏幕尺寸。与几年前不同，目前的手机屏幕已经大到一只手握不下了。标配早已经到了5寸屏时代。

所谓屏幕尺寸指的是屏幕对角线的长度，单位是英寸。

然而不同的屏幕尺寸是可以采用相同的分辨率的，而它们之间的区别在与密度（density）不同。

下面先介绍一下密度的概念，DPI、PPI，最后讲解一下如何根据获得的 Display 信息去求出屏幕尺寸。这是一个困扰我很久的问题了。

## 三、屏幕密度

屏幕密度与 DPI 这个概念紧密相连，DPI 全拼是 dots-per-inch,即每英寸的点数。也就是说，密度越大，每英寸内容纳的点数就越多。
android.util 包下有个 DisplayMetrics 类可以获得密度相关的信息。

最重要的是 densityDpi 这个成员，它有如下几个常用值：

```
    DENSITY_LOW = 120  
    DENSITY_MEDIUM = 160  //默认值  
    DENSITY_TV = 213      //TV专用  
    DENSITY_HIGH = 240  
    DENSITY_XHIGH = 320  
    DENSITY_400 = 400  
    DENSITY_XXHIGH = 480  
    DENSITY_XXXHIGH = 640  
```

举例如下：

```
    private void getDensity() {  
        DisplayMetrics displayMetrics = getResources().getDisplayMetrics();  
        Log.d(TAG,"Density is "+displayMetrics.density+" densityDpi is "+displayMetrics.densityDpi+" height: "+displayMetrics.heightPixels+  
            " width: "+displayMetrics.widthPixels);  
    }  
```

Log 如下：

```
    the screen size is Point(1600, 2438)  
    the screen real size is Point(1600, 2560)  
    Density is 2.0 densityDpi is 320 height: 2438 width: 1600  
```

有了这些信息，我们是不是就可以计算屏幕尺寸了呢？

首先求得对角线长，单位为像素。

然后用其除以密度（densityDpi）就得出对角线的长度了。

代码如下：

```
    private void getScreenSizeOfDevice() {  
        DisplayMetrics dm = getResources().getDisplayMetrics();  
        int width=dm.widthPixels;  
        int height=dm.heightPixels;  
        double x = Math.pow(width,2);  
        double y = Math.pow(height,2);  
        double diagonal = Math.sqrt(x+y);  
    
        int dens=dm.densityDpi;  
        double screenInches = diagonal/(double)dens;  
        Log.d(TAG,"The screenInches "+screenInches);  
    }  
```

Log 如下：

```
    01-13 16:35:03.026  16601-16601/com.linc.listviewanimation D/MainActivity﹕ the screen size is Point(1600, 2438)  
    01-13 16:35:03.026  16601-16601/com.linc.listviewanimation D/MainActivity﹕ the screen real size is Point(1600, 2560)  
    01-13 16:35:03.026  16601-16601/com.linc.listviewanimation D/MainActivity﹕ Density is 2.0 densityDpi is 320 height: 2438 width: 1600 xdpi 338.666 ydpi 338.666  
    01-13 16:35:03.026  16601-16601/com.linc.listviewanimation D/MainActivity﹕ The screenInches 9.112922229586951  
```

如 Log 所见，使用 heightPixels 得出的值是2483而不是正确的2560.从而使结果9.11反倒跟真实屏幕尺寸很接近。下面用正确的 height 再算一遍。

```
    01-13 16:39:05.476  17249-17249/com.linc.listviewanimation D/MainActivity﹕ the screen size is Point(1600, 2560)  
    01-13 16:39:05.476  17249-17249/com.linc.listviewanimation D/MainActivity﹕ the screen real size is Point(1600, 2560)  
    01-13 16:39:05.476  17249-17249/com.linc.listviewanimation D/MainActivity﹕ Density is 2.0 densityDpi is 320 height: 2560 width: 1600 xdpi 338.666 ydpi 338.666  
    01-13 16:39:05.476  17249-17249/com.linc.listviewanimation D/MainActivity﹕ The screenInches 9.433981132056605  
```

结果是9.43英寸，而真实值是8.91.如果再换一个设备，那么值差的更多。说明上面的计算是错误的。

那么错在哪里呢？densityDpi 是每英寸的点数（dots-per-inch）是打印机常用单位（因而也被称为打印分辨率），而不是每英寸的像素数。下面引出 PPI 这个概念。

## 四、PPI

Pixels per inch，这才是我要的每英寸的像素数（也被称为图像的采样率）。有了这个值，那么根据上面的公式就可以求导出屏幕的物理尺寸了。
还好 DisplayMetrics 有两个成员是 xdpi 和 ydpi，对其描述是：

```
    /The exact physical pixels per inch of the screen in the X/Y dimension.  
```

屏幕 X/Y 轴上真正的物理 PPI。

Yes！Got it！

为了保证获得正确的分辨率，我还是使用 getRealSize 去获得屏幕宽和高像素。所以，经过修改，代码如下：

```
    private void getScreenSizeOfDevice2() {  
        Point point = new Point();  
        getWindowManager().getDefaultDisplay().getRealSize(point);  
        DisplayMetrics dm = getResources().getDisplayMetrics();  
        double x = Math.pow(point.x/ dm.xdpi, 2);  
        double y = Math.pow(point.y / dm.ydpi, 2);  
        double screenInches = Math.sqrt(x + y);  
        Log.d(TAG, "Screen inches : " + screenInches);  
    }  
```

Log is as follows:

```
    [plain] view plaincopy在CODE上查看代码片派生到我的代码片
    01-13 16:58:50.142  17249-17249/com.linc.listviewanimation D/MainActivity﹕ Screen inches : 8.914015757534717  
```

## 五、DIP

注意不要与上面的 DPI 混淆，这个 DIP 是 Density Independent Pixel，直译为密度无关的像素。

我们在布局文件中使用的 dp/dip 就是它。官方推荐使用 dp 是因为它会根据你设备的密度算出对应的像素。

公式为：pixel = dip*density

需要注意的是，我们在 Java 代码中对控件设置宽高是不可以设置单位的，而其自带的单位是像素。所以如果动态修改控件大小时，我们的任务就来了，那就是将像素转换为 dp。

实例代码如下：

```
    //pixel = dip*density;  
    private int convertDpToPixel(int dp) {  
        DisplayMetrics displayMetrics = mContext.getResources().getDisplayMetrics();  
        return (int)(dp*displayMetrics.density);  
    }  
  
    private int convertPixelToDp(int pixel) {  
        DisplayMetrics displayMetrics = mContext.getResources().getDisplayMetrics();  
        return (int)(pixel/displayMetrics.density);  
    }   
```

参考：

http://stackoverflow.com/questions/19155559/how-to-get-android-device-screen-size

