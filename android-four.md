#[Android 实战技巧之四：语音识别](http://blog.csdn.net/lincyang/article/details/7463364)

Google 的语音识别是有目共睹的，所以 Android 上面也是沾了大光了，用起来简单至极。

过程如下：

1、启动语音识别 Activity

2、这里处理语音（传到 google 服务器处理）

3、结果以 Acitivity 的结果返回（onActivityResult）

主要用到的类为 [android.speech.RecognizerIntent](http://developer.android.com/reference/android/speech/RecognizerIntent.html)

下面的例子参考了 API Demo。

```
    package com.linc;  
  
    import java.util.ArrayList;  
    import java.util.List;  
  
    import android.app.Activity;  
    import android.content.Intent;  
    import android.content.pm.PackageManager;  
    import android.content.pm.ResolveInfo;  
    import android.os.Bundle;  
    import android.speech.RecognizerIntent;  
    import android.view.View;  
    import android.view.View.OnClickListener;  
    import android.widget.Button;  
    import android.widget.TextView;  
  
    public class VoiceRecognitionDemoActivity extends Activity {  
        private static final String TAG = "VoiceRecognition";  
        private static final int VOICE_RECOGNITION_REQUEST_CODE = 1234;  
        
        private TextView textView;  
        private Button button;  
        @Override  
        public void onCreate(Bundle savedInstanceState) {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.main);  
          
            initWidget();  
           
            // Check to see if a recognition activity is present  
            PackageManager pm = getPackageManager();  
            List<ResolveInfo> activities = pm.queryIntentActivities(  
                    new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH), 0);  
            if (activities.size() != 0) {  
                button.setOnClickListener(new OnClickListener() {  
                    @Override  
                    public void onClick(View v) {  
                        startVoiceRecognitionActivity();  
                    }  
                });  
            } else {  
                button.setEnabled(false);  
                button.setText("Recognizer not present");  
            }  
        }  
      
        private void initWidget()  
        {  
            textView = (TextView)findViewById(R.id.tv);  
            button = (Button)findViewById(R.id.btn);  
        }  
      
        /** 
         * Fire an intent to start the speech recognition activity. 
         */  
        private void startVoiceRecognitionActivity() {  
            Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);  
   
            // Display an hint to the user about what he should say.  
            intent.putExtra(RecognizerIntent.EXTRA_PROMPT, "请说标准普通话");//注意不要硬编码  
  
            // Given an hint to the recognizer about what the user is going to say  
            intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,  
                    RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);  
  
            // Specify how many results you want to receive. The results will be sorted  
            // where the first result is the one with   higher confidence.  
            intent.putExtra(RecognizerIntent.EXTRA_MAX_RESULTS, 5);//通常情况下，第一个结果是最准确的。  
  
            startActivityForResult(intent, VOICE_RECOGNITION_REQUEST_CODE);  
        }  
      
        @Override  
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {  
            if (requestCode == VOICE_RECOGNITION_REQUEST_CODE && resultCode == RESULT_OK) {  
                // Fill the list view with the strings the recognizer thought it could have heard  
                ArrayList<String> matches = data.getStringArrayListExtra(  
                        RecognizerIntent.EXTRA_RESULTS);  
                StringBuilder stringBuilder = new StringBuilder();  
                int Size = matches.size();   
                for(int i=0;i<Size;++i)  
                {  
                    stringBuilder.append(matches.get(i));  
                    stringBuilder.append("\n");  
                }  
                textView.setText(stringBuilder);  
            }  
  
            super.onActivityResult(requestCode, resultCode, data);  
       }  
    }  
```

结论：

在 wifi（家里的，1 兆带宽）状态下，识别速度飞快。

语音包是买手机时预装好的。

识别率很高。中文、英文、数字都能很好的识别。

用手机的 gprs，效果不尽如人意。几次识别都用了30秒左右的时间，体验很糟。