# [Android 实战技巧之一：文本与布局](http://blog.csdn.net/lincyang/article/details/7259322)

//别看这个标题挺大，其实这次要说的只是3个小技巧。
//2014.11.7 update

## **1、字符串资源里变量替换**

工作中是拒绝硬编码的，android 里会把一些字符串等放在 xml 中当做资源使用，如项目中 values 下的 strings.xml 列出了 app_name.
有些情况是这样的,程序中要使用的字符串如第345页,345是变量,那么我们不可能用两个字符串资源如

```
    <string name="di">第</string>  
    <string name="page">页</string> 
```
 
在程序中拼接字符串。那么我们可能会想，在我们学习 C 的时候，有 %s 这样的神奇的符号，可以替换变量的格式化操作符。

其实，在 android 中也有这样的东西，那就是 XLIFF，全称叫 XML 本地化数据交换格式，英文全称 XML Localization Interchange File Format。

用法也是很简单的，如

```
    <string name="page">第%1$s页</string>  
```

程序中只要给变量赋值就可以了，如

```
    String page = getString(R.string.page,"345");  
```

那么，要是有多个变量呢，如第345页24行？这也好办，如下：

```
   <string name="page">第%1$s页%2$s行</string>  
```

```
    String page = getString(R.string.page,"345","24");
```

## 2、**TextView 中设置多种字体大小**

这是项目中经常遇到的，比如 UI 是这样的：

Android 实战技巧之文本与布局

像这样的两种字体，要如何处理呢？需要用到 android.text 命名空间下的一些与 spannable 相关的类和接口。例子如：  

```
    String text = "Android实战技巧之文本与布局";  
    int start = text.indexOf('之');  
    int end = text.length();  
    Spannable textSpan = new Spannable(text);  
    textSpan.setSpan(new AbsoluteSizeSpan(20),0,start,Spannable.SPAN_INCLUSIVE_INCLUSIVE);  
    textSpan.setSpan(new AbsoluteSizeSpan(12),start,end,Spannable.SPAN_INCLUSIVE_INCLUSIVE); 
```

这个 textSpan 就是你想要的。

## 3、TextView 的超链接

这个很简单，在 xml 中属性 autoLink=“all”。

程序中 TextView.setAutoLink(Linkify.ALL);

说下参数：

Linkify.EMAIL_ADDRESS -- 仅识别出 TextView 中的 Email 在址，标识为超链接，点击后会跳到 Email，发送邮件给此地址

Linkify.PHONE_NUMBERS -- 仅识别出 TextView 中的电话号码，标识为超链接，点击后会跳到 Dialer，Call 这个号码

Linkify.WEB_URLS-- 仅识别出 TextView 中的网址，标识为超链接，点击后会跳到 Browser 打开此 URL

Linkify.ALL -- 这个选项是识别出所有系统所支持的特殊 Uri，然后做相应的操作

**特殊情况：**

当一段文字部分是超链接或者我们需要点击超链接跳到另一个 Activity时，如何处理？

答案还是用 Spannable。例子如下（摘自网络）： 

```
    public class MainActivity extends Activity {  
        private TextView testText;  
        @Override  
        protected void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.activity_main);  
            testText = (TextView) findViewById(R.id.testText);  
            //将TextView的显示文字设置为SpannableString  
            testText.setText(getClickableSpan());  
            //设置该句使文本的超连接起作用  
            testText.setMovementMethod(LinkMovementMethod.getInstance());  
        }  
  
        //设置超链接文字  
        private SpannableString getClickableSpan(){  
            SpannableString spanStr = new SpannableString("使用该软件，即表示您同意该软件的使用条款和隐私政策");  
            //设置下划线文字  
            spanStr.setSpan(new UnderlineSpan(), 16, 20,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
            //设置文字的单击事件  
            spanStr.setSpan(new ClickableSpan() {  
  
                @Override  
                public void onClick(View widget) {  
  
                    startActivity(new Intent (MainActivity.this, TestActivity1.class));  
                }  
            }, 16, 20, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  
            //设置文字的前景色  
            spanStr.setSpan(new ForegroundColorSpan(Color.WHITE), 16, 20,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  
            //设置下划线文字  
            spanStr.setSpan(new UnderlineSpan(), 21, 25,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);    
            //设置文字的单击事件  
            spanStr.setSpan(new ClickableSpan() {  
              
                @Override  
                public void onClick(View widget) {  
                  
                    startActivity(new Intent(MainActivity.this, TestActivity2.class));  
                }  
            }, 21, 25, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  
            //设置文字的前景色  
            spanStr.setSpan(new ForegroundColorSpan(Color.WHITE), 21, 25,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);  
            return spanStr;  
        }  
    }
```

## 4、布局中只在界面底部

在大部分的 android 程序中，都会发现一个底部的菜单栏。这通常是一个线性布局加一些按钮。如何让其始终在底部，无论是哪个分辨率呢？
这要用到关系布局的属性

```
    android:layout_alignParentBottom="true"  
```

在关系布局内部，如果把此属性设置 true，就会在关系布局的底部了。
这个用途还是很广泛的。

## 5.EditText 与软键盘

当界面有 EditText 并且光标落在上面时，软键盘就会弹出。本来是为了方便，但有些情况这样挺讨厌的。比如登录界面。

取消它只需要在 Manifest 文件中使用 windowSoftInputMode 即可，如下：

```
    <activity   
        android:name=".LoginActivity"  
        android:label="@string/app_name"  
        android:windowSoftInputMode="stateHidden|adjustResize"  
        >  
```

## 6.布局的边框颜色

// 2014.11.24 updated

尝试一下用各种 layout 仿制 listview，就是把 layout 的边框设置对应的颜色。

在drawable下添加layer_list，

```
    <?xml version="1.0" encoding="utf-8"?>  
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">  
      <item>  
        <shape>  
            <stroke android:width="3px" android:color="#ffff0000" /><!--边框颜色-->  
            <solid android:color="#FFFCFCFC" /><!--填充色-- >  
            <corners android:radius="4dp" /><!--圆角-->  
        </shape>  
      </item>  
    </layer-list>  
```

在 layout 中引用：

``` 
    <LinearLayout  
        android:layout_width="match_parent"  
        android:layout_height="300dp"  
        android:orientation="vertical"  
        android:background="@drawable/layout_bg"  
        >  
```
//2014.11.27 update 

其他 widget 如 ImageView 的边框也可以像这样设置。

## 7.Java 文件中字体加粗

```
    //2015.1.12 update  
    //Typeface  
    textView.setTypeface(Typeface.defaultFromStyle(Typeface.BOLD));  
    //use TextPaint  
    textView.getPaint().setFakeBoldText(true);  
```




