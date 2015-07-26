#[Android 实战技巧之五：读取 Excel](http://blog.csdn.net/lincyang/article/details/16982245)

最近有这么个需求，发现有现成的开源库 jxl 可以完美实现我的小需求。
参考两篇文章：

[利用 Java 操作 Excel](http://www.ibm.com/developerworks/cn/java/l-javaExcel/)

[官方 blog 教程](http://www.andykhan.com/jexcelapi/tutorial.html)

源码：<I>jexcelapi.sourceforge.net/</I>

直接练习一下，用 Javac 编译：

```
    1 import java.io.*;  
    2 import jxl.*;  
    3   
    4 public class Test  
    5 {  
    6         public static void main(String[] args)  
    7         {  
    8                 try {  
    9                 InputStream is = new FileInputStream("test.xls");  
    10                 Workbook book = Workbook.getWorkbook(is);  
    11                 Sheet sheet = book.getSheet(0);  
    12                 Cell cell = sheet.getCell(2,2);  
    13                 String result = cell.getContents();  
    14                 System.out.println(result);  
    15                 book.close();  
    16                 }  
    17                 catch(Exception e) {  
    18                 System.out.println(e);  
    19                 }  
    20            
    21         }  
    22 }   
```

居然报错：

```
    linc:~/workspace/java/test-excel$ javac -cp jxl.jar Test.java   
    linc:~/workspace/java/test-excel$ ls  
    jxl.jar  Test.class  Test.java  test.xls  
    linc:~/workspace/java/test-excel$ java Test  
    Exception in thread "main" java.lang.NoClassDefFoundError: jxl/Workbook  
        at Test.main(Test.java:9)  
    Caused by: java.lang.ClassNotFoundException:  jxl.Workbook  
        at java.net.URLClassLoader$1.run(URLClassLoader.java:202)  
        at java.security.AccessController.doPrivileged(Native Method)  
        at java.net.URLClassLoader.findClass(URLClassLoader.java:190)  
        at java.lang.ClassLoader.loadClass(ClassLoader.java:306)  
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)  
        at java.lang.ClassLoader.loadClass(ClassLoader.java:247)  
        ... 1 more  
```

网上搜寻方法，并没有解决此问题，没办法，直接用 Eclipse 来做练习，正常的加入 JARS 就可以了。

练习的代码如下，读取 Excel 内容并显示在 textview 中，textview 可以上下滚动。

大概代码如下：

main.xml

```
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
        xmlns:tools="http://schemas.android.com/tools"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"  
        android:paddingBottom="@dimen/activity_vertical_margin"  
        android:paddingLeft="@dimen/activity_horizontal_margin"  
        android:paddingRight="@dimen/activity_horizontal_margin"  
        android:paddingTop="@dimen/activity_vertical_margin"  
        tools:context=".MainActivity" >  
  
        <TextView  
            android:id="@+id/txt_show"  
            android:layout_width="wrap_content"  
            android:layout_height="wrap_content"  
            android:singleLine="false"  
            android:scrollbars="vertical"  
            android:text="@string/hello_world" />  
  
    </RelativeLayout>  
```

MainActivity.java

```
    package com.linc.readdata;  
  
    import java.io.FileInputStream;  
    import java.io.InputStream;  
  
    import android.os.Bundle;  
    import android.app.Activity;  
    import android.text.method.ScrollingMovementMethod;  
    import android.view.Menu;  
    import android.widget.TextView;  
  
    import jxl.*;  
  
    public class MainActivity extends Activity {  
        TextView txt = null;  
      
        @Override  
        protected void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.activity_main);  
            txt = (TextView)findViewById(R.id.txt_show);  
            txt.setMovementMethod(ScrollingMovementMethod.getInstance());  
            readExcel();  
        }  
  
        @Override  
        public boolean onCreateOptionsMenu(Menu menu) {  
            // Inflate the menu; this adds items to the action bar if it is present.  
            getMenuInflater().inflate(R.menu.main, menu);  
            return true;  
        }  
       
        public void readExcel() {  
              try {  
               InputStream is = new FileInputStream("mnt/sdcard/test.xls");  
               //Workbook book = Workbook.getWorkbook(new File("mnt/sdcard/test.xls"));  
               Workbook book = Workbook.getWorkbook(is);  
               int num = book.getNumberOfSheets();  
               txt.setText("the num of sheets is " + num+  "\n");  
               // 获得第一个工作表对象  
               Sheet sheet = book.getSheet(0);  
               int Rows = sheet.getRows();  
               int Cols = sheet.getColumns();  
               txt.append("the name of sheet is " + sheet.getName() + "\n");  
               txt.append("total rows is " + Rows + "\n");  
               txt.append("total cols is " + Cols + "\n");  
               for (int i = 0; i < Cols; ++i) {  
                for (int j = 0; j < Rows; ++j) {  
                 // getCell(Col,Row)获得单元格的值  
                    txt.append("contents:" + sheet.getCell(i,j).getContents() + "\n");  
                }  
               }  
               book.close();  
              } catch (Exception e) {  
               System.out.println(e);  
              }  
            }  
  
   }  
```

完整项目（带jxl.jar）请猛击[这里](http://download.csdn.net/detail/lincyang/6618417)。

而其他操作本人并没有验证，请自行去读官方教程或参考以下网络中的实现：

```
    public void createExcel() {  
      try {  
       // 创建或打开Excel文件  
       WritableWorkbook book = Workbook.createWorkbook(new File(  
         "mnt/sdcard/test.xls"));  
       // 生成名为“第一页”的工作表,参数0表示这是第一页  
       WritableSheet sheet1 = book.createSheet("第一页",  0);  
       WritableSheet sheet2 = book.createSheet("第三页",  2);  
       // 在Label对象的构造函数中,元格位置是第一列第一行(0,0)以及单元格内容为test  
       Label label = new Label(0, 0, "test");  
       // 将定义好的单元格添加到工作表中  
       sheet1.addCell(label);  
       /* 
        * 生成一个保存数字的单元格.必须使用Number的完整包路径,否则有语法歧义 
        */  
       jxl.write.Number number = new jxl.write.Number(1, 0, 555.12541);  
       sheet2.addCell(number);  
       // 写入数据并关闭文件  
       book.write();  
       book.close();  
      } catch (Exception e) {  
       System.out.println(e);  
      }  
    }  
    /** 
      * jxl暂时不提供修改已经存在的数据表,这里通过一个小办法来达到这个目的,不适合大型数据更新! 这里是通过覆盖原文件来更新的. 
      * 
      * @param filePath 
      */  
    public void updateExcel(String filePath) {  
      try {  
      Workbook rwb = Workbook.getWorkbook(new File(filePath));  
       WritableWorkbook wwb = Workbook.createWorkbook(new File(  
        "d:/new.xls"), rwb);// copy  
       WritableSheet ws = wwb.getSheet(0);  
       WritableCell wc = ws.getWritableCell(0, 0);  
       // 判断单元格的类型,做出相应的转换  
       Label label = (Label) wc;  
       label.setString("The value has been modified");  
       wwb.write();  
       wwb.close();  
       rwb.close();  
       } catch (Exception e) {  
       e.printStackTrace();  
      }  
    }  
    public static void writeExcel(String filePath) {  
      try {  
       // 创建工作薄  
       WritableWorkbook wwb = Workbook.createWorkbook(new  File(filePath));  
       // 创建工作表  
       WritableSheet ws = wwb.createSheet("Sheet1", 0);  
       // 添加标签文本  
       // Random rnd = new Random((new Date()).getTime());  
       // int forNumber = rnd.nextInt(100);  
       // Label label = new Label(0, 0, "test");  
       // for (int i = 0; i < 3; i++) {  
       // ws.addCell(label);  
       // ws.addCell(new jxl.write.Number(rnd.nextInt(50), rnd  
       // .nextInt(50), rnd.nextInt(1000)));  
       // }  
       // 添加图片(注意此处jxl暂时只支持png格式的图片)  
       // 0,1分别代表x,y 2,5代表宽和高占的单元格数  
       ws.addImage(new WritableImage(5, 5, 2, 5, new File(  
         "mnt/sdcard/nb.png")));  
       wwb.write();  
       wwb.close();  
      } catch (Exception e) {  
       System.out.println(e.toString());  
      }  
    }  
    }  
```‎