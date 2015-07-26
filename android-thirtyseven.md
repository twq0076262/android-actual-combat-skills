#[Android 实战技巧之三十七：图片的 Base64 编解码](http://blog.csdn.net/lincyang/article/details/46596899)

[`base64`](http://www.csdn.net/tag/base64) [`encode`](http://www.csdn.net/tag/encode) [`decode`](http://www.csdn.net/tag/decode) [`bitmap`](http://www.csdn.net/tag/bitmap)

通常用 Base64 这种编解码方式将二进制数据转换成可见的字符串格式，就是我们常说的大串，10 块钱一串的那种，^_^。

Android 的 android.util 包下直接提供了一个功能十分完备的 Base64 类供我们使用，下面就演示一下如何将一张图片进行 Base64 的编解码。

**1.找到那张图片**

```

    public void onEncodeClicked(View view) {

            //select picture
            Intent intent = new Intent();
            intent.setType("image/*");
            intent.setAction(Intent.ACTION_GET_CONTENT);
            startActivityForResult(intent, OPEN_PHOTO_FOLDER_REQUEST_CODE);
        }

        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            if(OPEN_PHOTO_FOLDER_REQUEST_CODE == requestCode   && RESULT_OK == resultCode) {

               //encode the image
                Uri uri = data.getData();
                try {
                    //get the image path
                    String[] projection = {MediaStore.Images.Media.DATA};
                    CursorLoader cursorLoader = new CursorLoader(this,uri,projection,null,null,null);
                    Cursor cursor = cursorLoader.loadInBackground();
                    int column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
                    cursor.moveToFirst();

                    String path = cursor.getString(column_index);
                    Log.d(TAG,"real path: "+path);
                    encode(path);
                } catch (Exception ex) {
                    Log.e(TAG, "failed." + ex.getMessage());
                }
            }
        }

```

**2.将图片转换成 bitmap 并编码**

```

    private void encode(String path) {
                    //decode to bitmap
                    Bitmap bitmap = BitmapFactory.decodeFile(path);
                    Log.d(TAG, "bitmap width: " + bitmap.getWidth() + " height: " + bitmap.getHeight());
                    //convert to byte array
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    bitmap.compress(Bitmap.CompressFormat.PNG, 100, baos);
                    byte[] bytes = baos.toByteArray();

                    //base64 encode
                    byte[] encode = Base64.encode(bytes,Base64.DEFAULT);
                    String encodeString = new String(encode);
                    mTvShow.setText(encodeString);
    }

```

**3.将大串还原成图片**

```

    public void onDecodeClicked(View view) {
        byte[] decode = Base64.decode(mTvShow.getText().toString(),Base64.DEFAULT);
        Bitmap bitmap = BitmapFactory.decodeByteArray(decode, 0, decode.length);
        //save to image on sdcard
        saveBitmap(bitmap);
    }

    private void saveBitmap(Bitmap bitmap) {
        try {
            String path = Environment.getExternalStorageDirectory().getPath()
                    +"/decodeImage.jpg";
            Log.d("linc","path is "+path);
            OutputStream stream = new FileOutputStream(path);
            bitmap.compress(Bitmap.CompressFormat.JPEG, 90, stream);
            stream.close();
            Log.e("linc","jpg okay!");
        } catch (IOException e) {
            e.printStackTrace();
            Log.e("linc","failed: "+e.getMessage());
        }
    }

```

需要注意的是，一张图片的编码速度会很慢，如果图片很大就更慢了。毕竟手机的处理能力有限。不过 decode 的速度确实相当的快，超出你的想象。好了，就是这样简单，今天就到这里了，晚安！