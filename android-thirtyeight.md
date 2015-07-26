#[Android 实战技巧之三十八：Handler 使用中可能引发的内存泄漏](http://blog.csdn.net/lincyang/article/details/46875157)

[`Handler`](http://www.csdn.net/tag/Handler) [`Looper`](http://www.csdn.net/tag/Looper) [`MessageQue`](http://www.csdn.net/tag/MessageQue) [`内存泄漏`](http://www.csdn.net/tag/%e5%86%85%e5%ad%98%e6%b3%84%e6%bc%8f)

<table class="table table-bordered table-striped table-condensed"> <tr> <td>目录(?)[+]</td> </tr> </table>

##问题描述

曾几何时，我们用原来的办法使用 Handler 时会有下面一段温馨的提示：

```

    This Handler class should be static or leaks might occur

```

下面是更详细的说明（Android Studio上的警告，不知道 Eclipse 上是否相同）

Since this Handler is declared as an inner class, it may prevent the outer class from being garbage collected. If the Handler is using a Looper or MessageQueue for a thread other than the main thread, then there is no issue. If the Handler is using the Looper or MessageQueue of the main thread, you need to fix your Handler declaration, as follows: Declare the Handler as a static class; In the outer class, instantiate a WeakReference to the outer class and pass this object to your Handler when you instantiate the Handler; Make all references to members of the outer class using the WeakReference object.

大概意思就是：

一旦 Handler 被声明为内部类，那么可能导致它的外部类不能够被垃圾回收。如果 Handler 是在其他线程（我们通常成为 worker thread）使用 Looper 或 MessageQueue（消息队列），而不是 main 线程（UI 线程），那么就没有这个问题。如果 Handler 使用 Looper 或 MessageQueue 在主线程（main thread），你需要对 Handler 的声明做如下修改： 

声明 Handler 为 static 类；在外部类中实例化一个外部类的 WeakReference（弱引用）并且在 Handler 初始化时传入这个对象给你的 Handler；将所有引用的外部类成员使用 WeakReference 对象。

##解决方案一

上面的描述中基本上把推荐的修改方法明确表达了出来，下面的代码是我自己使用中的一个实现，请参考：

```

    private CopyFileHandler mHandler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_appstart);
        mHandler = new CopyFileHandler(this);
        startCopyDBThread();
    }

    private void startCopyFileThread(){
        Log.d(TAG, "startCopyDBThread");
        new Thread(new Runnable() {
            @Override
            public void run() {
                //DO SOMETHING LIKE: copyDBFile();
                Message msg=mHandler.obtainMessage();
                mHandler.sendMessage(msg);
            }
        }).start();
    }

    private static class CopyFileHandler extends Handler {
        WeakReference<AppStartActivity> mActivity;
        public CopyFileHandler(AppStartActivity activity)   {
            mActivity = new WeakReference<>(activity);
        }

        public void handleMessage(Message msg) {
            final AppStartActivity activity = mActivity.get();
            //handle you message here!
        }
    }

```

##为什么会内存泄漏

那么为什么不这样做会引发内存泄漏呢？ 

这与几个关键词有关：内部类、Handler 的消息循环（Looper）、Java 垃圾回收机制。 

需要强调一下，并不是每次使用 Handler 都会引发内存泄漏，这里面有一定的几率，需要满足特定条件才会引起泄漏。
 
**内部类会有一个指向外部类的引用。** 

**垃圾回收机制中约定，当内存中的一个对象的引用计数为 0 时，将会被回收。**
 
**Handler 作为 Android 上的异步消息处理机制（好吧，我大多用来进行 worker thread 与 UI 线程同步），它的工作是需要 Looper 和 MessageQueue 配合的。简单的说，要维护一个循环体（Looper）处理消息队列（MessageQueue）。每循环一次就从 MessageQueue 中取出一个 Message，然后回调相应的消息处理函数。**

如果，我是说如果，循环体中有消息未处理（Message 排队中），那么 Handler 会一直存在，那么 Handler 的外部类（通常是 Activity）的引用计数一直不会是 0，所以那个外部类就不能被垃圾回收。很多人会遇到 activity 的 onDestroy 方法一直不执行就是这个原因。

##另一个解决方案的尝试

警告描述中提到了 Handler在worker thread 中使用 Looper 或 MessageQueue，我尝试了一下，请大家品鉴。

```

    private Handler testHandler;
    private Thread mThread = new Thread() {
        public void run() {
            Log.d(TAG,"mThread run");
            Looper.prepare();
            testHandler = new Handler() {
                public void handleMessage(Message msg) {
                    Log.d("TAG", "worker thread:"+Thread.currentThread().getName());
                    switch (msg.what) {
                        //handle message here
                    }
                }
            };
            Looper.loop();
        }
    };

    //start thread here
    if(Thread.State.NEW == mThread.getState()) {
        Log.d(TAG, "mThread name: " + mThread.getName());
        mThread.start();
    }

    //send message here
    testHandler.sendEmptyMessage(1);

```

参考： 

[http://stackoverflow.com/questions/11407943/this-handler-class-should-be-static-or-leaks-might-occur-incominghandler](http://stackoverflow.com/questions/11407943/this-handler-class-should-be-static-or-leaks-might-occur-incominghandler)

[http://m.blog.csdn.net/blog/wurensen/41907663](http://m.blog.csdn.net/blog/wurensen/41907663) 

[http://blog.csdn.net/lmj623565791/article/details/38377229](http://blog.csdn.net/lmj623565791/article/details/38377229)