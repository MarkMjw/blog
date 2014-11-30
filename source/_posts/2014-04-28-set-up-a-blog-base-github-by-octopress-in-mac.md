---
title: "在Mac环境下通过Octopress搭建自己的Blog并托管到Github"
date: 2014-04-28 14:02
comments: true
tags: ['Octopress', 'Github']
---

一直都想有一个自己的博客，CSDN以及 OSChina这种技术论坛提供的博客系统在国内也算不错的了，但是使用过后总觉得不够自由，没有个性，各种被束缚的感觉。偶然间发现了[Wordpress](https://wordpress.org/)以及[Octopress](http://octopress.org/)，瞬间眼前一亮，看到国内外各路大神都有一个自己的博客，瞬间燃起了自己动手搞一个博客的想法，经过各种挫折与磨难，总算初步搞定，顺便根据参考的各种资料记录下搭建步骤，一方面害怕某天自己就忘记了，另一方面也给后来的童鞋一个参考。本文涉及到[Git](http://git-scm.com/)的基本使用以及[Github](https://github.com/)相关知识，如果还不知道这俩货是啥的童鞋，我表示有必要恶补一下了，基本的东西不再赘述，不清楚的童鞋请自行Google。

<!--more-->

## 1.安装环境
因为`Octopress`需要通过`Git`进行部署，且需要`Ruby`运行环境，所以需要安装`Git`和`Ruby`，且`Ruby`版本必须为`1.9.3`以上，通常`osx`系统已经安装好了`Git`以及`Ruby`，可以通过以下命令来查看相关信息：

``` bash
$ git --version
$ ruby --version
```

如果都已安装且符合要求则可以直接跳至第二步。没有安装的推荐使用大名鼎鼎的[Homebrew](http://brew.sh/index_zh-cn.html)来进行安装。

### 安装brew

由于`MacPorts`与`Homebrew`不兼容所以需要先卸载`MacPorts`，终端执行如下命令：

``` bash
$ sudo prot -f uninstall installed
$ sudo rm -fr \
```

接下来再通过以下命令安装`Homebrew`：

``` bash
$ curl -L http://github.com/mxcl/homebrew/tarball/master | tar xz –strip 1 -C /usr/local
$ export PATH=/usr/local/bin:$PATH
```

安装成功后通过`brew --version`查看brew版本，接下来就可以通过`Homebrew`来安装`Git`以及`Ruby`了。

### 安装Git
安装`Git`很简单只需在终端执行一行命令：

``` bash
$ brew install git
```

### 安装Ruby
安装`Ruby`只需在终端下依次执行一下命令即可：

``` bash
$ brew install rbenv
$ brew install ruby-build
$ rbenv install
$ rbenv rehash
```

## 2.安装Octopress
相关的准备工作完成之后，现在正是开始搭建博客了，首先是安装`Octopress`。

### 安装Octopress
只需要要再终端执行以下命令：

``` bash
$ git clone https://github.com/imathis/octopress.git octopress
$ cd octopress
$ gem install bundler
$ rbenv rehash
$ bundle install
$ rake install
```

### 配置Octopress
安装完成之后，需要做一些简单地配置工作，编辑`_config.yml`文件的`url`,`title`,`subtitle`,`author`。

这里可以寻找一份配置好的文件直接进行修改，[参考我的配置文件](https://github.com/MarkMjw/blog_jekyll/blob/master/_config.yml)。

### 安装支持新浪微博和Dribbble的Octopress的Greyshade主题
我现在用的也是greyshade主题，执行如下命令：
``` bash
$ cd octopress
$ git clone https://github.com/shashankmehta/greyshade.git .themes/greyshade
$ echo "\$greyshade: color;" >> sass/custom/_colors.scss // 替换 color 为自定义的链接高亮颜色
$ rake "install[greyshade]"
```

在`_config.yml`中加入

```
weibo_user: xxx #微博UID
dribbble_user:
weibo_share: true #是否开启微博分享按钮
```

关于`Greyshade`主题的头像问题，有两种途径可以设置头像

* 在`_config.yml`文件中设置一个`email`，然后到[Gravatar](https://en.gravatar.c om/)网站上添加该`email`并上传一张头像
* 将需要使用的图片放到`/source/images`下。然后把``/source/_includes/header.html`文件中的`img`替换成 `img alt=“Profile Picture” src=“/images/tx.png” style=“width:160px;”`

### 配置Disqus插件
[Disqus](https://disqus.com/)是`Octopress`内置的评论功能，编辑`_config.yml`文件可以打开该功能，找到以下内容修改

```
#Disqus comments
disqus_short_name: #在Disqus设置的short name
disqus_show_comment_count: false
```

填入注册`Disqus`账号设置的`short name`，并将`false`修改为`true`。 **Disqus要和自己的username.github.com关联上**

## 3.配置Github相关
### 在本机创建ssh

```bash
$ cd ~/.ssh
$ ssh-keygen -t rsa -C <你注册github时的email>
```

弹出`Enter file in which to save the key (/Users/twer/.ssh/id_rsa): `直接按空格

弹出`Enter passphrase (empty for no passphrase): `输入你Github账号的密码。`Enter same passphrase again: `再次输入你的密码。

打开`~/.ssh`下的`id_rsa.pub`文件复制里面的全部内容。
登陆`Github`，选择 **Account Settings-->SSH Public Keys**  添加`ssh`，把剪切板的内容复制到`key`输入框内直接保存。

测试shh:

```bash
$ ssh git@github.com
```

输出

```
PTY allocation request failed on channel 0
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

代表成功

### 建立一个仓库
登陆Github[创建一个仓库](https://github.com/new) ，仓库名称为`username.github.io`比如我的是`markmjw.github.io`。


## 4.部署博客到Github
利用`Octopress`的一个配置`rake`任务来自动配置上面创建的仓库：可以让我们方便的部署`GitHub pages`。在终端输入如下命令：

```bash
$ rake setup_github_pages
```

弹出之后输入`https://github.com/yourusername/yourusername.github.io.git`，我的是`https://github.com/MarkMjw/markmjw.github.io.git`


**输入以下命令部署博客**

```bash
$ rake generate
$ rake deploy
```

如果无法push到仓库的master分支，尝试在项目目录的`.git/config`中添加
```bash
[branch "master"]
remote = origin
merge = refs/heads/master
```

如果在`deploy`时出错，错误信息如：

```
## Pushing generated _deploy website
To https://github.com/MarkMjw/username.github.io.git
! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/username/	username.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

则进行手动push：

```bash
$ cd _deploy
$ git push -f #强制push，这会直接清空远程仓库原来的内容
```

博客的`source`目录可以单独提交，执行如下命令就可以将`source`提交到仓库的`source`分支下

```bash
$ git add --all
$ git commit -m 'Initial source commit'
$ git push origin source
```

部署前可以在本地预览，输入`rake preview`之后在浏览器输入`http://localhost:4000/`来访问


## 5.写博客
通过命令

```bash
$ rake new_post["My first blog"]
```

文章生成在目录下的`source/_posts`目录下。文章是`markdown`格式的。可以通过 [Mou](http://mouapp.com) 来编辑保存。

关于`Markdown`的语法可以参考这篇文章:[http://wowubuntu.com/markdown/](http://wowubuntu.com/markdown/)

写完后就可以部署更新文章到`Github`上了
```bash
#build and deploy
$ rake generate
$ rake deploy

#commit sources
$ git add --all
$ git commit -am "Some comment here."
$ git push origin source
```

**参考文章**

* <http://www.sgxiang.com/blog/2013/10/18/cong-ling-kai-shi-da-jian-ji-yu-githubde-octopressbo-ke/>
