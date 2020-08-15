# OkHttp

## 配置

1. 添加依赖

   ```java
   implementation 'com.squareup.okhttp3:okhttp:4.4.0'
   ```

2. 添加联网权限

   ```java
   <uses-permission android:name="android.permission.INTERNET" />
   ```

## GET

### 使用步骤

1. 拿到OkHttpClient对象

   ```java
   OkHttpClient client = new OkHttpClient();
   ```

2. 构造Request对象

   ```java
   Request request = new Request.Builder()
                   .get()
                   .url("")
                   .build();
   ```

3. 将Request封装为Call

   ```java
   Call call = client.newCall(request);
   ```

4. 根据需要调用同步或者异步请求方法

   ```java
   //同步调用,返回Response,会抛出IO异常
   Response response = call.execute();
   
   //异步调用,并设置回调函数
   call.enqueue(new Callback() {
       @Override
       public void onFailure(Call call, IOException e) {
           Toast.makeText(OkHttpActivity.this, "get failed", Toast.LENGTH_SHORT).show();
       }
   
       @Override
       public void onResponse(Call call, final Response response) throws IOException {
           final String res = response.body().string();
           runOnUiThread(new Runnable() {
               @Override
               public void run() {
                   contentTv.setText(res);
               }
           });
       }
   });
   ```

### 下载文件

#### 从网络下载一个文件

```java
public void downloadImg(View view){
    OkHttpClient client = new OkHttpClient();
    final Request request = new Request.Builder()
            .get()
            .url("https://www.baidu.com/img/bd_logo1.png")
            .build();
    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {
            Log.e("moer", "onFailure: ");;
        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            //拿到字节流
            InputStream is = response.body().byteStream();

            int len = 0;
            File file  = new File(Environment.getExternalStorageDirectory(), "n.png");
            FileOutputStream fos = new FileOutputStream(file);
            byte[] buf = new byte[128];

            while ((len = is.read(buf)) != -1){
                fos.write(buf, 0, len);
            }

            fos.flush();
            //关闭流
            fos.close();
            is.close();
        }
    });
}
```

#### 从网络下载一张图片并设置到ImageView中

```java
@Override
public void onResponse(Call call, Response response) throws IOException {
    InputStream is = response.body().byteStream();

    final Bitmap bitmap = BitmapFactory.decodeStream(is);
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            imageView.setImageBitmap(bitmap);
        }
    });

    is.close();
}
```

## POST

### 提交键值对

1. 拿到OkHttpClient对象

   ```java
   OkHttpClient client = new OkHttpClient();
   ```

2. 构建FormBody,传入参数

   ```java
   FormBody formBody = new FormBody.Builder()
                   .add("username", "admin")
                   .add("password", "admin")
                   .build();
   ```

3. 构建Request,将FormBody作为Post方法的参数传入

   ```java
   final Request request = new Request.Builder()
                   .url("http://www.jianshu.com/")
                   .post(formBody)
                   .build();
   ```

4. 将Request封装为Call

   ```java
   Call call = client.newCall(request);
   ```

5. 调用请求,重写回调方法

   ```java
   call.enqueue(new Callback() {
       @Override
       public void onFailure(Call call, IOException e) {
           Toast.makeText(OkHttpActivity.this, "Post Failed", Toast.LENGTH_SHORT).show();
       }
   
       @Override
       public void onResponse(Call call, Response response) throws IOException {
           final String res = response.body().string();
           runOnUiThread(new Runnable() {
               @Override
               public void run() {
                   contentTv.setText(res);
               }
           });
       }
   });
   ```

### 提交字符串

```java
RequestBody requestBody = RequestBody.create(MediaType.parse("text/plain;charset=utf-8"), "{username:admin;password:admin}");
```

### 提交表单

1. 添加依赖

   ```java
   compile 'com.squareup.okio:okio:1.11.0'
   ```

2. 提交表单

   ```java
   File file = new File(Environment.getExternalStorageDirectory(), "1.png");
   if (!file.exists()){
       Toast.makeText(this, "文件不存在", Toast.LENGTH_SHORT).show();
       return;
   }
   RequestBody muiltipartBody = new MultipartBody.Builder()
           //一定要设置这句
           .setType(MultipartBody.FORM)
           .addFormDataPart("username", "admin")//
           .addFormDataPart("password", "admin")//
           .addFormDataPart("myfile", "1.png", RequestBody.create(MediaType.parse("application/octet-stream"), file))
           .build();
   ```

### 上传文件

1. 添加存储卡写权限

   ```java
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
   ```

2. 上传文件

   ```java
   File file = new File(Environment.getExternalStorageDirectory(), "1.png");
   if (!file.exists()){
       Toast.makeText(this, "文件不存在", Toast.LENGTH_SHORT).show();
   }else{
       RequestBody requestBody2 = RequestBody.create(MediaType.parse("application/octet-stream"), file);
   }
   ```

## 参考文献

[OkHttp使用详解](https://bingjian.blog.csdn.net/article/details/53728818)