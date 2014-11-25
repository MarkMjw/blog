---
layout: post
title: "快速构建Android REST客户端 1"
date: 2014-04-27 20:21:44 +0800
comments: true
categories: Android
---

![](/media/2014-04-27-building-android-rest-client-applications-efficiently/Driibo_on_MX2.jpg)

日常使用的各种客户端：新浪微博客户端、Google+、Path、人人等，都属于REST App，如何快速的构建`REST`客户端也成为了近年来`Google IO`大会上的热门课题，而我在这方面也投入了近一年的经历（从[项目经历][1]来看，简直就是客户端专业户了，泪···）。下面我将以最近开发的[Driibo(on github)][2]为例，手把手带大家开发`REST`客户端。
<!--more-->

- - -
应用架构
===
![](/media/2014-04-27-building-android-rest-client-applications-efficiently/architecture.png)
- - -
网络通讯
===
原始的HttpClient开发成本较高，最好使用更为先进的库，比如以下这些：

* [Volley][3] Google于2013 Google io大会上发布的网络通讯库，`Google Play`,`Driibo`均基于此库构建
* [AsyncHttpClient][4] SonaType的一个开源库，移植到Android平台需要进行一些定制，
* [Retrofit][5] Square的开源库，于`Volley`相比`Retrofit`更专注于网络请求和数据解析，贵在专一。


`Volley`在提供了高性能网络通讯功能的同时，对网络图片加载也提供了良好的支持，完全可以满足简单`REST`客户端的需求。

###使用Volley
下载`Volley`源码
``` bash
$ git clone https://android.googlesource.com/platform/frameworks/volley
```

由于`Volley`直接提供了源码，我们必须自己build jar包，在`Volley`项目根目录下执行
``` bash
$ ant jar
```
然后将所得的Jar包导入到工程中。

###请求队列
为了统一管理请求，我在RequestManager.java中维持了一个请求队列实例。
``` java
    public static RequestQueue mRequestQueue = newRequestQueue();
    private static RequestQueue newRequestQueue() {
        RequestQueue requestQueue = new RequestQueue(openCache(), new BasicNetwork(new HurlStack()));
        requestQueue.start();
        return requestQueue;
    }
```
值得注意的是，`Volley`提供了中断特定请求的方法，利用这个特性可以在`Activity`关闭时中断该页面的请求，节省有限资源的消耗（节流省电，顶Google）。
``` java 
    // tag是中断请求所需的token
    public static void addRequest(Request request, Object tag) {
        if (tag != null) {
            request.setTag(tag);
        }
        mRequestQueue.add(request);
    }

    public static void cancelAll(Object tag) {
        mRequestQueue.cancelAll(tag);
    }
```


###构建图片缓存
由于`Volley`提供了本地缓存，在`Driibo`中我只需要实现一个内存缓存:
``` java
package com.refactech.driibo.data;

import com.android.volley.toolbox.ImageLoader;
import com.refactech.driibo.util.ImageUtils;

import android.graphics.Bitmap;
import android.support.v4.util.LruCache;

/**
 * Created by Issac on 7/19/13.
 */
public class BitmapLruCache extends LruCache<String, Bitmap> implements ImageLoader.ImageCache {

    public BitmapLruCache(int maxSize) {
        super(maxSize);
    }

    @Override
    protected int sizeOf(String key, Bitmap bitmap) {
        return ImageUtils.getBitmapSize(bitmap);
    }

    @Override
    public Bitmap getBitmap(String url) {
        return get(url);
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        put(url, bitmap);
    }
}

```
`Driibo`使用了一个全局的`BitmapLruCache`，取内存阈值的1/3作为图片缓存的大小，这样在界面跳转的时不必释放图片资源，更有效的利用内存并防止`OOM`:
``` java
 // 取运行内存阈值的1/3作为图片缓存
    private static final int MEM_CACHE_SIZE = 1024 * 1024 * ((ActivityManager) AppData.getContext()
            .getSystemService(Context.ACTIVITY_SERVICE)).getMemoryClass() / 3;

    private static DiskBasedCache mDiskCache = (DiskBasedCache) mRequestQueue.getCache();
```

###图片加载
`Volley`提供了`ImageLoader`,`NetworkImageView`用于处理图片加载。其中`NetworkImageView`主要用于非滚动视图的图片加载，可直接替换ImageView。
对于列表中的图片加载我在`Driibo`中是这样处理的：

1. 创建ImageLoader实例
``` java
 private static ImageLoader mImageLoader = new ImageLoader(mRequestQueue, new BitmapLruCache(
            MEM_CACHE_SIZE));
```

2. 在ShotsAdapter中加载图片
```java
  @Override
    public void bindView(View view, Context context, Cursor cursor) {
        Holder holder = getHolder(view);
        if (holder.imageRequest != null) {
            holder.imageRequest.cancelRequest();
        }
        ...

        Shot shot = Shot.fromCursor(cursor);
        holder.imageRequest = RequestManager.loadImage(shot.getImage_url(), RequestManager
                .getImageListener(holder.image, mDefaultImageDrawable, mDefaultImageDrawable));
...
}
```
必须保存请求实例，并在列表滚动时取消上次的请求，以防止列表中图片加载错乱的情况。

**Be Careful:** 切不可在列表中混用`NetworkImageView`和`ImageLoader`，否则会出现图像无法加载的情况，原因未查证。。。

- - -
相关演讲
===
* [Google I/O 2010 - Android REST client applications](http://www.youtube.com/watch?v=xHXn3Kg2IQE)
* [Google I/O 2013 - Volley: Easy, Fast Networking for Android](http://www.youtube.com/watch?v=yhv8l9F44qo)

需<del>翻墙</del>

- - -
下一章
===
[快速构建Android REST客户端 2](/blog/2014/04/27/building-android-rest-client-applications-efficiently-number-2/)

[1]: /project
[2]: https://github.com/Issacw0ng/Dribbo
[3]: https://android.googlesource.com/platform/frameworks/volley
[4]: https://github.com/AsyncHttpClient/async-http-client
[5]: http://square.github.io/retrofit/