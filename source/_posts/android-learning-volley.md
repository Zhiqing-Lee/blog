---
title: Android学习笔记：volley初试
date: 2016-07-24
categories:
- Android
tags:
- Android
- Code
- Volley
---

Volley是一个Android的HTTP库，用于方便地执行网络操作。

Volley不适用与下载很大的文件。

## 开始使用

### Clone 仓库源码到本地(需翻墙)：

```bash
git clone https://android.googlesource.com/platform/frameworks/volley
```

### 导入Volley

1. 在Android Studio中选择`File` -> `New` -> `Import Module`
2. 在弹出的对话框中的`Source directory`栏中选择刚刚Clone到本地的文件夹
3. 在Module name中输入模块名（默认为`:volley`)

![volley](/images/android-import-volley.png)

### 添加依赖

- 代码方式：在`build.gradle (Module: app)`文件中的`dependencies`部分加入`compile project(':volley')`(其中`:volley`为导入模块时输入的名字)：

```groovy
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile project(':volley')
}
```

- 在 Android Studio 中选择 `File` -> `Project Structure`，在`Modules`中选中`app`，然后选择`Dependencies`，单击右边绿色的`+`，选择`Module dependency`，选择刚刚导入的模块(`:volley`)。

### 添加INTERNET权限

在`manifest`文件中添加如下代码：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## 请求队列

使用 Volley 的方式是，创建一个 RequestQueue 并传递 Request 对象给它。RequestQueue 管理用来执行网络操作的工作线程、从缓存中读取数据、写数据到缓存、并解析 Http 的响应内容。Volley 会把解析完的响应数据分发给主线程。

### 使用Volley.newRequestQueue

Volley 提供了一个便捷方法 `Volley.newRequestQueue()` 来创建 RequestQueue：

```java
TextView mTextView = (TextView) findViewById(R.id.text_view);

// 创建一个RequestQueue
RequestQueue queue = Volley.newRequestQueue(this);
String url ="http://baidu.com";

// 创建一个StringRequest，请求响应结果为String
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
            new Response.Listener() {
    @Override
    public void onResponse(String response) {
        // 显示响应结果到一个TextView
        mTextView.setText(response);
    }
}, new Response.ErrorListener() {
    @Override
    public void onErrorResponse(VolleyError error) {
        //当遇到错误时显示'Error!'
        mTextView.setText("Error!");
    }
});
// 添加该请求到请求队列
queue.add(stringRequest);
```

### 创建自定义请求队列

#### 设置网络和缓存

RequestQueue需要两部分来支持工作：网络(Network)和缓存(Cache)：

```java
RequestQueue mRequestQueue;

// 创建一个缓存
Cache cache = new DiskBasedCache(getCacheDir(), 1024 * 1024);

// 设置网络使用HttpURLConnection类作为HTTP客户端。
Network network = new BasicNetwork(new HurlStack());

// 用创建的缓存和网络来实例化一个请求队列
mRequestQueue = new RequestQueue(cache, network);

// 开启队列
mRequestQueue.start();

String url ="http://www.example.com";

StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
        new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // Do something with the response
    }
},
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            // Handle error
    }
});

// 添加请求到请求队列
mRequestQueue.add(stringRequest);
```

#### 使用单例模式

一个提供了RequestQueue和ImageLoader的单例类：

```java
public class MySingleton {
    private static MySingleton mInstance;
    private RequestQueue mRequestQueue;
    private ImageLoader mImageLoader;
    private static Context mCtx;

    private MySingleton(Context context) {
        mCtx = context;
        mRequestQueue = getRequestQueue();

        mImageLoader = new ImageLoader(mRequestQueue,
                new ImageLoader.ImageCache() {
            private final LruCache<String, Bitmap>
                    cache = new LruCache<String, Bitmap>(20);

            @Override
            public Bitmap getBitmap(String url) {
                return cache.get(url);
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {
                cache.put(url, bitmap);
            }
        });
    }

    public static synchronized MySingleton getInstance(Context context) {
        if (mInstance == null) {
            mInstance = new MySingleton(context);
        }
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            // getApplicationContext() is key, it keeps you from leaking the
            // Activity or BroadcastReceiver if someone passes one in.
            mRequestQueue = Volley.newRequestQueue(mCtx.getApplicationContext());
        }
        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req) {
        getRequestQueue().add(req);
    }

    public ImageLoader getImageLoader() {
        return mImageLoader;
    }
}
```

一个单例类的使用例子：

```java
// 获取一个请求队列
RequestQueue queue = MySingleton.getInstance(this.getApplicationContext()).
    getRequestQueue();

// 添加一个请求到请求队列
MySingleton.getInstance(this).addToRequestQueue(stringRequest);
```

### 取消请求

1. 为Request设置标签：

  ```java
  public static final String TAG = "MyTag";

  // 为请求设置标签
  stringRequest.setTag(TAG);

  // 添加请求到请求队列
  mRequestQueue.add(stringRequest);
  ```

2. 在Activiy的onStop()方法中取消包含该标签的请求：

  ```java
  @Override
  protected void onStop () {
      super.onStop();
      if (mRequestQueue != null) {
          mRequestQueue.cancelAll(TAG);
      }
  }   
  ```

## 请求

### Request生命周期

![volley生命周期](/images/volley-request.png)

### 创建标准请求

Volley 的几种种常用的请求：

- `StringRequest`: 指定一个URL并且接收字符串类型的响应数据
- `ImageRequset`: 指定一个URL并且接收图片类型的响应数据
- `JsonObjectRequest`和`JsonArrayRequest`: 指定一个URL并且接收Json对象(数组)类型的响应数据

#### 请求图片

##### 使用ImageRequset

```java
ImageView mImageView;
String url = "http://zhiqing.info/images/logo.png";
mImageView = (ImageView) findViewById(R.id.image_view);

// 根据URL请求图片并显示到用户界面
ImageRequest request = new ImageRequest(url,
    new Response.Listener() {
        @Override
        public void onResponse(Bitmap bitmap) {
            mImageView.setImageBitmap(bitmap);
        }
    }, 0, 0, null,
    new Response.ErrorListener() {
        public void onErrorResponse(VolleyError error) {
            mImageView.setImageResource(R.drawable.image_load_error);
        }
    });
// 通过单例类取得RequestQueue并将图片请求添加到RequestQueue
MySingleton.getInstance(this).addToRequestQueue(request);
```

##### 使用 ImageLoader 和 NetworkImageView

使用ImageLoader显示图片：

```java
ImageLoader mImageLoader;
ImageView mImageView;
// The URL for the image that is being loaded.
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
mImageView = (ImageView) findViewById(R.id.regularImageView);

// Get the ImageLoader through your singleton class.
mImageLoader = MySingleton.getInstance(this).getImageLoader();
mImageLoader.get(IMAGE_URL, ImageLoader.getImageListener(mImageView,
         R.drawable.def_image, R.drawable.err_image));
```

使用NetworkImageView：

```xml
<com.android.volley.toolbox.NetworkImageView
    android:id="@+id/networkImageView"
    android:layout_width="150dp"
    android:layout_height="170dp"
    android:layout_centerHorizontal="true" />
```

```java
ImageLoader mImageLoader;
NetworkImageView mNetworkImageView;
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";

// Get the NetworkImageView that will display the image.
mNetworkImageView = (NetworkImageView) findViewById(R.id.networkImageView);

// Get the ImageLoader through your singleton class.
mImageLoader = MySingleton.getInstance(this).getImageLoader();

// Set the URL of the image that should be loaded into this view, and
// specify the ImageLoader that will be used to make the request.
mNetworkImageView.setImageUrl(IMAGE_URL, mImageLoader);
```

#### 请求Json

```java
TextView mTxtDisplay;
ImageView mImageView;
mTxtDisplay = (TextView) findViewById(R.id.txtDisplay);
String url = "http://my-json-feed";

JsonObjectRequest jsObjRequest = new JsonObjectRequest
        (Request.Method.GET, url, null, new Response.Listener() {

    @Override
    public void onResponse(JSONObject response) {
        mTxtDisplay.setText("Response: " + response.toString());
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        // TODO Auto-generated method stub

    }
});

// Access the RequestQueue through your singleton class.
MySingleton.getInstance(this).addToRequestQueue(jsObjRequest);
```
