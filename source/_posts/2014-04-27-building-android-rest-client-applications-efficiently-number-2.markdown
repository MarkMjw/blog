---
layout: post
title: "快速构建Android REST客户端 2"
date: 2014-04-27 20:30:09 +0800
comments: true
categories: Android
---

数据解析和存储
===
###JsonPaser
如果你还在使用`Android`提供的`JsonObject`来解析JSON数据，是时候鸟枪换炮了！
Android开发中常用的JsonPaser有这么几个：

* [Gson][1] 由Google开发的Json Parser 使用简单，只需要引入一个jar包，相对于`Android`系统提供的parser，效率有明显的优势
* [Jackson][2] 另一个颇有名气的项目，需要引入很多包相对比较复杂
* [FastJson][3] 阿里主导的开源项目，要求类的每一个属性都具备getter/setter这一点我不能忍
<!--more-->

Gson的使用方法异常简单，例如我们要解析以下数据：
``` json
String json ="{
        "id": 1,
        "name": "Dan Cederholm",
        "username": "simplebits",
        "url": "http://dribbble.com/simplebits",
        "avatar_url": "http://dribbble.com/system/users/1/avatars/original/dancederholm-peek.jpg",
        "location": "Salem, MA",
        "twitter_screen_name": "simplebits",
        "drafted_by_player_id": null,
        "shots_count": 147,
        "draftees_count": 103,
        "followers_count": 2027,
        "following_count": 354,
        "comments_count": 2001,
        "comments_received_count": 1509,
        "likes_count": 7289,
        "likes_received_count": 2624,
        "rebounds_count": 15,
        "rebounds_received_count": 279,
        "created_at": "2009/07/07 21:51:22 -0400"
    }"
```
需要创建Player类，在该类中声明所有需要解析的属性：
``` java
public class Player {
    private long id;
    private String name;
    private String url;
    private String avatar_url;
    private String location;
    private String twitter_screen_name;
    private String drafted_by_player_id;
    private int shots_count;
    private int draftees_count;
    private int followers_count;
    private int following_count;
    private int comments_count;
    private int comments_received_count;
    private int likes_count;
    private int likes_received_count;
    private int rebounds_count;
    private int rebounds_received_count;
    private String created_at;
}
```
使用Gson:
``` java
Gson gson = new Gson();
// Json to object
gson.fromJson(json, Player.class);
// Object to json
String json2 = gson.toJson(Player.class);
```

本着“不会偷懒的程序员不是好程序员”这一遵旨，在`Driibo`的设计中，数据库中仅存储了Json数据，免去了分列的繁琐。缺点是读取数据时需要用Gson重新解析，对效率有一定影响。


###ContentProvider
`ContentProvider`着实让人很头痛，实际上Android文档中提到，如果没有跨进程的需求，或者向其他应用分享数据的需求就不必使用ContentProvider。但`ContentProvider`为数据库的管理提供了更清晰的接口，并且为了使用`CursorLoader`，`ContentProvider`是必须构建的。

更多细节还请参阅 [Api Guides - Content Providers][4]

###CursorAdapter & CursorLoader
使用`CursorLoader`是目前在`Activity`&`Fragment`中异步读取`ContentProvider`的最佳方案。CursorLoader还有一个更为强劲的功能，就是在接到数据变更的通知时会重新query一次，这样就保证了Cursor的数据始终与数据库同步。

基于以上两个特性，在列表中使用CursorAdapter填充数据可以有效的简化数据加载和更新的逻辑。

更多细节参阅 [API Guides - Loaders][5]


[1]: https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CCsQFjAA&url=http%3a%2f%2fcode%2egoogle%2ecom%2fp%2fgoogle-gson%2f&ei=Vv3sUengEoPJkwXcvYDIDg&usg=AFQjCNGGFFMez8-PfFoEQP93a7eHFY8ssA
[2]: http://jackson.codehaus.org/
[3]: http://code.alibabatech.com/wiki/display/FastJSON/Home-zh
[4]: http://developer.android.com/guide/topics/providers/content-providers.html
[5]: http://developer.android.com/guide/components/loaders.html