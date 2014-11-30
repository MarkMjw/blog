---
title: MIUI V5中系统级AlertDialog无法显示
date: 2014-09-17 14:44
comments: true
tags: ['Android']
---

在开发过程中相信很多人都有过：在`Service`或者后台Manager中弹出`Dialog`而不依赖于某个特定的`Activity`这样的需求。通常的做法是将`Dialog`对应的`WindowManager.LayoutParams`的`type`设置为`TYPE_SYSTEM_ALERT`，这样就解决了在`Service`中弹出`Dialog`的需求。
<!--more-->

然而，这样做只对原生为经过修改的`ROM`有效，而对`MIUI`这种修改过的`ROM`来说就不兼容了。找了许久才发现罪魁祸首竟然是`MIUI`默认关闭了每个应用显示系统悬浮窗的功能。如图：

![小米3截屏](/media/2014-09-17-the-system-dialog-don-not-show-in-miui/MI3-screen-shot.png)

## 解决办法
### 1.手动打开显示悬浮窗的功能

  具体设置路径：

```
系统设置->应用->对应的APP主页
```

不过这似乎对用户而言是解决了问题，但是对于开发者而言如何帮助用户解决呢？
部分产品的解决办法是在APP中增加对`MIUI`系统用户的引导，让其在使用时手动打开此功能。

### 2.程序根据ROM自动判断并兼容

#### 分两种情况：

* 只需要在本应用内显示
* 系统全局显示

针对第一种情况：我的做法是首先在`Application`中保存所有启动的`Activity`的实例，然后在需要显示`Dialog`的时候进行判断如是`MIUI`系统则通过`Application`获取当前`Activity`的实例来进行显示`Dialog`。

判断是否是`MIUI`系统代码如下：
``` java
    public static boolean isMIUI() {
        String systemProperty = getSystemProperty("ro.miui.ui.version.name");
        return !TextUtils.isEmpty(systemProperty);
    }

    public static String getSystemProperty(String propName) {
        String line;
        BufferedReader input = null;
        try {
            Process p = Runtime.getRuntime().exec("getprop " + propName);
            input = new BufferedReader(new InputStreamReader(p.getInputStream()), 1024);
            line = input.readLine();
            input.close();
        } catch (IOException ex) {
            LogUtil.e(TAG, "Unable to read sysprop " + propName, ex);
            return null;
        } finally {
            if (input != null) {
                try {
                    input.close();
                } catch (IOException e) {
                    LogUtil.e(TAG, "Exception while closing InputStream", e);
                }
            }
        }
        return line;
    }
```

第二种情况相对复杂一些，我目前也没有找到一种好的解决办法，也许只能提示用户进行手动打开权限了。
