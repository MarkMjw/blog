---
title: MIUI无法显示ActionMode
date: 2014-09-17 17:03
comments: true
tags: ['Android']
---

最近项目中遇到一个特别恶心的事情，在做`ListView`多选[Selection][1]的功能时使用了系统的[ActionMode][2]功能，大部分的手机上功能都很正常
唯独在MIUI上始终无法显示且没有任何错误提示。各种Google之后仍然找不到原因，无奈之下只好自己挨个测试，花了近乎一天的时间总算找到点眉目。

![小米2截屏](/media/2014-09-17-miui-dont-show-actionmode/mi2-screen-shot-01.png)

<!--more-->

### 原因

在`menu`的布局文件中将`item`设置如下：
        android:showAsAction="always"
此时在MIUI中，长按`ListView`虽然会进入多选的状态但`ActionMode`无法显示。但如将`item`的设置为：
        android:showAsAction="never"
则可以显示，但由于产品的要求该`item`需要始终显示，该怎么办呢？冥思许久，既然不能直接搞定，那么我就采取曲线救国的方式，解决办法如下。

### 解决办法

非常幸运Google为我们提供了非常强大的`ActionBar`，可以通过方法设置自定义View:
        mActionBar.setCustomView(R.layout.vw_action_layout);
这样我们只需在`vw_action_layout`中按照各自的需求进行布局即可，然后按照一般的额View设置各自的监听事件即可。

### 最终效果
![小米2截屏](/media/2014-09-17-miui-dont-show-actionmode/mi2-screen-shot-02.png)

[1]: http://developer.android.com/design/patterns/selection.html
[2]: http://developer.android.com/reference/android/view/ActionMode.html
