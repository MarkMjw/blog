---
title: Good bye Octopress，Helo Hexo
date: 2014-11-25 12:48
comments: true
tags: ['Hexo', 'Octopress']
---

好吧，我承认我又犯贱了，这次从`Octopress`到`Hexo`。对于我来说`Octopress`有个致命的缺点：**框架略显复杂，折腾起来太过于麻烦**。之前几乎花了两天的时间才基本搞定，然而过了一段时间之后又出了一些乱七八糟的问题，又得各种折腾。无意间看到了`Hexo`，瞬间被其简洁快速的特点所打动，于是毫不犹豫的决定转至`Hexo`。

## Hexo是什么
[Hexo](http://hexo.io/)是一个基于[Node.js](http://nodejs.org/)的静态博客程序，类似于[Jekyll](http://jekyllrb.com/)、[Octopress](http://octopress.org/)、[Wordpress](http://cn.wordpress.org/)，我们可以使用[Hexo](http://hexo.io/)创建自己的博客，托管到[Github](https://github.com)或[Heroku](http://www.heroku.com)上，绑定自己的域名，用[Markdown](http://zh.wikipedia.org/wiki/Markdown)写文章。作者是来自台湾的[@tommy351](https://github.com/hexojs/hexo),引用[@tommy351](https://github.com/hexojs/hexo)的话：

>一个快速、简单且功能强大的 Node.js 博客框架。
>A fast, simple & powerful blog framework, powered by Node.js.

<!--more-->

## 为什么要用Hexo

>静态文件生成速度快
>支持 Markdown 编写文章
>易用，平时写文章只需掌握四个命令，部署更简单仅需一道指令即可，不想Jekyll需要很多命令
>已移植很多 Octopress 插件
>高扩展性、自订性
>兼容于 Windows, Mac & Linux
>简洁，文件少，方便理解，相对于Jekyll来说体积小了不少

## 搭建Hexo博客

### 1.安装Git

* Windows: 下载安装[Msysgit](http://msysgit.github.io/).
* Mac: 通过这几种方式安装 [Homebrew](http://brew.sh/index_zh-cn.html), [MacPorts](http://www.macports.org/) 或者 [installer](https://code.google.com/p/git-osx-installer/).
* Linux (Ubuntu, Debian): sudo apt-get install git-core
* Linux (Fedora, Red Hat, CentOS): sudo yum install git-core

### 2.安装Node.js

安装 [Node.js](http://nodejs.org/) 非常简单，仅须[下载](http://nodejs.org/download/)安装文件并执行即可完成安装。  

### 3.安装Hexo

利用`npm`命令即可安装.

``` bash
$ npm install -g hexo
```

安装完成后，运行以下命令`Hexo`会在你指定的目录下生成所有需要的文件.

``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

### 4.本地预览

现在我们已经搭建起本地的`Hexo`博客了，在刚才指定的目录下进入终端执行以下命令，然后到浏览器输入`localhost:4000`看看.

``` bash
$ hexo generate
$ hexo server
```

### 5.部署到Github

没有`Github`账号请在此进行[注册](https://github.com/signup/free)，然后创建一个新的`repository`，名字应该是：`xxx.github.io`.
编辑`_config.yml`(你指定的Hexo目录下)如下：

    deploy:
        type: github
        repository: https://github.com/your_name/your_name.github.io.git
        branch: master


### 6.创建新文章并部署

* 创建一个新的文章

``` bash
$ hexo new "My New Post"
```

* 启动服务

``` bash
$ hexo server
```

* 生成静态文件

``` bash
$ hexo generate
```

* 部署到远程服务器

``` bash
$ hexo deploy
```

更多信息请查看官方[文档](http://hexo.io/docs).
