# [Android 实战技巧之二十八：启动另一个 App/apk 中的 Activity](http://blog.csdn.net/lincyang/article/details/45503675)

Android 提供了在一个 App 中启动另一个 App 中的 Activity 的能力，这使我们的程序很容易就可以调用其他程序的功能，从而就丰富了我们App的功能。比如在微信中发送一个位置信息，对方可以点击这个位置信息启动腾讯地图并导航。这个场景在现实中作用很大，尤其是朋友在陌生的环境找不到对方时，这个功能简直就是救星。

本来想把本文的名字叫启动另一个进程中的 Activity，觉得这样才有逼格。因为每个 App 都会运行在自己的虚拟机中，每个虚拟机跑在一个进程中。但仔细一想，能够称为一个进程，前提是这个 App 必须要运行起来才行。而 Android 提供的能力，是不需要另一个 App 启动就可以将其特定的 Activity 启动起来的。

我们有至少两种办法达到启动另一个 App 中的 Activity，第一种用action启动，详情见我之前的文章《启动自己另一个程序的 activity》。

第二种用 intent 设置 className 或 component 的办法启动。举例如下。新建两个项目 ProjectA 和 ProjectB，用 B 中的 MainActivity 启动 A 的 MainActivitity。关键代码如下：
 
ProjectB MainActivity

```
       public void OnStartActivityClicked(View view) {
            Intent intent = new Intent(Intent.ACTION_VIEW);

            String packageName = "com.lazytech.projecta";
            String className = "com.lazytech.projecta.MainActivity";
            intent.setClassName(packageName, className);

            //second method
    //        intent.setComponent(new ComponentName(
    //                "com.lazytech.projecta",
    //                "com.lazytech.projecta.MainActivity"
    //        ));
            Bundle bundle = new Bundle();
            bundle.putString("msg", "this message is from project B ");
            intent.putExtras(bundle);
    
            intent.putExtra("pid", android.os.Process.myPid());

            startActivityForResult(intent, 1);
    //        startActivity(intent);
        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            switch (requestCode) {
                case 1:
                    if(resultCode == RESULT_OK) {
                        textView.setText(data.getStringExtra("result"));
                    }
                    break;
            }
        }
```

ProjectA MainActivity

```
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            textView = (TextView)findViewById(R.id.text);

            Intent intent = getIntent();
            if(intent != null) {
                textView.setText(intent.getStringExtra("msg"));
            }
        }

        public void OnSendResult(View view) {
            Intent intent = new Intent();
            intent.putExtra("result","OK! from project a.");
            this.setResult(RESULT_OK,intent);
            this.finish();
        }
```