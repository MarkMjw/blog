---
title: The complete lifecycle of Activity and Fragment
date: 2014-11-26 11:30:56
comments: true
tags: ['Android', 'Activity', 'Fragment']
---

`Activity`和`Fragment`在`Android`开发中使用非常频繁，我想很多人刚开始接触`Android`的时候首先被提到的就是四大组件吧，而首当其冲的便是`Activity`了，至少我是酱紫的。此外在学习`Activity`同时我们首先要掌握的便是`Activity`的 **生命周期** 了，开发过程中很多时候就是因为没有搞清楚`Activity` **生命周期** 的具体流程导致走了不少的弯路和遇到不少的Bug。`Fragment`是`Android3.0`推出的新的组件，与`Activity`非常类似，在项目中运用非常广泛，因此对它们 **生命周期** 的理解非常重要。

<!--more-->

首先来看看一直非常熟悉的图，来自官方[文档](http://developer.android.com/guide/components/activities.html)：

![Activity Liefcycle](/media/2014-11-26-Activity-and-Fragment-Lifecycle/activity_lifecycle.png)

想必大家对这张图应该非常的熟悉了吧，它将`Activity`的整个 **生命周期** 描述得非常清晰，接下来看看`Fragment`的 **生命周期** ，来自官方[文档](http://developer.android.com/guide/components/fragments.html)：

![Fragment Liefcycle](/media/2014-11-26-Activity-and-Fragment-Lifecycle/fragment_lifecycle.png)

看完这两张图应该对`Activity`和`Fragment`的 **生命周期** 有了一个整体认识了吧，但是总觉得哪里不够详细。幸好，国外开发者[Steve Pomeroy](https://github.com/xxv) 根据自己的理解和经验完成了一幅非常完善的 **生命周期图** 托管在[Github](https://github.com/xxv/android-lifecycle)，支持`dia`、`pdf`、`png`、`svg`等格式的下载，这里贴上原图：

![Android Liefcycle](/media/2014-11-26-Activity-and-Fragment-Lifecycle/complete_android_fragment_lifecycle.png)

虽然有了这么完善的文档，但是光看不练肯定是记不住的，所以想有深刻的理解并能灵活运用还得靠在实际项目中进行试验。
