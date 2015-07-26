#[Android 实战技巧之二十六：persistableMode 与 Activity 的持久化](http://blog.csdn.net/lincyang/article/details/45287599)

[`persistabl`](http://www.csdn.net/tag/persistabl) [`activity`](http://www.csdn.net/tag/activity) [`onsaveinst`](http://www.csdn.net/tag/onsaveinst)

API 21 为 Activity 增加了一个新的属性，只要将其设置成 persistAcrossReboots，activity 就有了持久化的能力，另外需要配合一个新的 bundle 才行，那就是 PersistableBundle。 

这里的持久化与传统意义的不同，它的具体实现在 Activity 重载的 onSaveInstanceState、onRestoreInstanceState 和 onCreate 方法。
 
```

    public void onSaveInstanceState(Bundle outState, PersistableBundle outPersistentState)
    
    public void onRestoreInstanceState(Bundle savedInstanceState, PersistableBundle persistentState)
    
    public void onCreate(Bundle savedInstanceState, PersistableBundle persistentState)

```

onSaveInstanceState 和 onRestoreInstanceState 方法是一对拯救灾难的方法，它们不在“正常“的 Activity 生命周期中，只有一些突发异常情况才会触发它们，比如横竖屏切换、按 Home 键等。当 API 21 后增加了 PersistableBundle 参数，令这些方法有了系统关机重启后数据恢复的能力。

网友们评价不一，但是无论如何这都为我们提供了一种便利。而它应用的场景是异常的状况，不会影响我们正常的数据持久化办法。比如在 pause 方法中做一些操作 Preferences，文件 I/O，SQLite 数据库，ContentProvider 等常规办法。

如何实践呢？ 

只需在 Manifest 中的 activity 设置属性：

```

    android:persistableMode="persistAcrossReboots"

```

然后在 activity 中直接用上述的三个方法即可。 

另外注意 API 版本是 21 及以上。

验证是个难题。因为我没有 5.0 及以上系统的设备，求助与模拟器吧，各种问题都来了。无论是 Genymotion 还是自带的模拟器，在关机的过程中模拟器都会卡死。虽然我在 log 里看到了程序已经走过了 onSaveInstanceState(Bundle,PersistableBundle)。就差模拟器关机后开启看效果。我这边是没有成功，如果哪位大虾看到了效果，请告诉我。