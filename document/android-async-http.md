## Android-Async-Http

[android-async-http的github地址](https://github.com/loopj/android-async-http)

[API地址](https://loopj.com/android-async-http/doc/)

### 一 基本介绍

android-async-http是基于Apache HttpClient专门用于android的异步http请求，所有的请求都在非UI线程执行。下面是基本特性:

* 采用异步http请求，并通过匿名内部类处理回调结果 
* http请求独立在UI主线程之外 
* 采用线程池来处理并发请求 
*  采用RequestParams类创建GET/POST参数 
* 不需要第三方包即可支持Multipart file文件上传 
* 大小只有25kb ；
* 自动为各种移动电话处理连接断开时请求重连 
* 使用BinaryHttpResponseHandler类下载二进制文件(如图片) 
* 使用JsonHttpResponseHandler类可以自动将响应结果解析为json格式 
* 持久化cookie存储，可以将cookie保存到你的应用程序的SharedPreferences中

### 二 集成

通过Maven将该库集成到项目中

```java
app/build.gradle文件中
repositories {
  mavenCentral()
}

dependencies {
  implementation 'com.loopj.android:android-async-http:1.4.9'
}
```



### 三 使用

#### 3.1 基本使用

##### 3.1.1 使用AsyncHttpResponseHandler

```java
AsyncHttpClient client = new AsyncHttpClient();
client.get("http://www.google.com", new AsyncHttpResponseHandler() {
    @Override
    public void onStart() {
        // called before request is started
    }
 
    @Override
    public void onSuccess(int statusCode, Header[] headers, byte[] response) {
        // called when response HTTP status is "200 OK"
    }

    @Override
    public void onFailure(int statusCode, Header[] headers, byte[] errorResponse, Throwable e) {
        // called when response HTTP status is "4XX" (eg. 401, 403, 404)
    }
 
    @Override
    public void onRetry(int retryNo) {
        // called when request is retried
    }
})
    
重写onStart、onFinish、onCancle、onRetry等方法来实现数据的获取以及与界面的交互效果
```

##### 3.1.2  使用RequestParams传递参数

```java
AsyncHttpClient client = new AsyncHttpClient();
RequestParams params = new RequestParams();
params.put("key", "value");
params.put("more", "data");
client.get("http://www.google.com", params, new
    AsyncHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Header[] headers, byte[] response） {
            System.out.println(response);
        }
 
        @Override
        public void onFailure(int statusCode, Header[] headers, byte[] responseBody, Throwable error） {
            Log.d("ERROR", error);
        }    
    }
);
```

##### 3.1.3 使用TextHttpResponseHandler

其继承自AsyncHttpResponse，并将原生的字节流根据指定的encoding转化成了string对象，

```java
AsyncHttpClient client = new AsyncHttpClient();
RequestParams params = new RequestParams();
params.put("key", "value");
params.put("more", "data");
client.get("http://www.google.com", params, new
    TextHttpResponseHandler() {
        @Override
        public void onSuccess(int statusCode, Header[] headers, String response） {
            System.out.println(response);
        }
 
        @Override
        public void onFailure(int statusCode, Header[] headers, String responseBody, Throwable error） {
            Log.d("ERROR", error);
        }    
    }
);
```

##### 3.1.4  使用JsonHttpResponseHandler

返回的response已经自动转化成JSONObject了，当然也支持JSONArray类型

```java
String url = "https://ajax.googleapis.com/ajax/services/search/images";
AsyncHttpClient client = new AsyncHttpClient();
RequestParams params = new RequestParams();
params.put("q", "android");
params.put("rsz", "8");
client.get(url, params, new JsonHttpResponseHandler() {            
    @Override
    public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
       // Handle resulting parsed JSON response here
    }
    @Override
    public void onSuccess(int statusCode, Header[] headers, JSONArray response) {
      // Handle resulting parsed JSON response here
    }
```

