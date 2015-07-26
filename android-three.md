#[Android 实战技巧之三：性能测试类](http://blog.csdn.net/lincyang/article/details/7457035)

[`测试`](http://www.csdn.net/tag/%e6%b5%8b%e8%af%95)  [`android`](http://www.csdn.net/tag/android) [`class`](http://www.csdn.net/tag/class) [`手机`](http://www.csdn.net/tag/%e6%89%8b%e6%9c%ba) [`Java`](http://www.csdn.net/tag/java)

通常来说手机上的程序都很金贵，配置不高但要良好的性能。虽然目前的新手机都有着显赫的配置，但性能方面仍然很重要。

Android 程序首推开发语言是 Java，易用的同时也带来了性能上的问题，尤其是在动画和游戏开发方面。

高性能高效率的程序也是很难求的，通常都是在几番磨难之后才能诞下这样的程序。

平时，我们应该多注意。不要以为性能离我们很远，其实它存在于我们的指尖。

下面是两个常用的测试类，一个是时间测试类，另一个是内存使用测试类。

经常测试一下效果，会得到意想不到的好处的。

大家不妨试试。

时间测试类

```

    package com.linc;  
  
  
    import android.util.Log;  
  
  
    public class TimeTest {  
        private static long startTime ;  
        private static long endTime ;  
        public static void start()  
        {  
            startTime = System.currentTimeMillis();  
        }  
        public static void end()  
        {  
            endTime = System.currentTimeMillis();  
            long time = endTime - startTime;  
            Log.i("TimeTest", "calculateProcessTime is "+time);  
        }  
    }  

```

内存测试类

```

    package com.linc;  
  
    import android.util.Log;  
   
    public class MemoryTest {  
        private static long startMemory;  
        private static long endMemory;  
      
        private static long memoryUsed()  
        {  
            long total = Runtime.getRuntime().totalMemory();  
            long free = Runtime.getRuntime().freeMemory();  
            return (total - free);  
        }  
      
        public static void start()  
        {  
            startMemory = memoryUsed();  
        }  
      
        public static void end()  
        {  
            endMemory = memoryUsed();  
            long memo = endMemory - startMemory;  
            Log.i("MemoryTest", "calculateUsedMemory is "+memo);  
        }  
    }

```  