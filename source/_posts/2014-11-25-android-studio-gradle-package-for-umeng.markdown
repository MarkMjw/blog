---
title: "Android Studio 通过Gradle打友盟渠道包"
date: 2014-11-25 10:09
tags: ['Android', 'Gradle', 'Android Studio', 'umeng']
---

几天前，Google终于发布了Android Studio 1.0RC 。从Google IO 2013发布至今历时一年半的时间，大大小小几十个版本的迭代总算熬出头了。从0.1.0的Preview到0.8.0的Beta再到1.0的Release Candidate，我几乎每一个版本都使用过，其中的辛酸相信用过的人都知道。前期的Android Studio不稳定，Bug很多，想要长期作为生产工具需要一定耐心，不过现在好了基本上趋于稳定了，当然问题依旧不少，出了问题网上也能搜到解决办法了。

Android 5.0正式版也于近期进行了推送，相信不少的 Nexus 死忠们已经升级了，我也不例外。不知道具体什么原因，所以友盟提供的打包工具打出来的包无法在Android 5.0上安装。所以无奈之下只好自己来写打包脚本了，虽然以前也写过Gradle的打包但是由于Google改动了不少东西导致不能使用了。接下来我会简单地记录一些关键点并加以说明。

<!--more-->

本文不会对Android Studio以及Gradle的基本使用做说明，仅针对已经有一定基础的朋友，Android Studio以及Gradle的基本知识请移步官方[文档](http://tools.android.com/tech-docs/new-build-system/user-guide)。

话不多说，首先来看看新版的Android Studio的Splash吧：
![Android Studio 最新闪屏](/media/2014-11-25-android-studio-gradle-package-for-umeng/splash.png)


### Manifest文件内容占位符

用过友盟统计的应该都知道，友盟渠道号其实通过实现定义在`AndroidManifest.xml`中`meta-data`值来进行区别的，所以我们需要通过Manifest文件占位符来进行设置。

1. 在`AndroidManifest`文件中定义一个占位符，格式如下，占位符名称随意。
```xml
    <meta-data
        android:name="UMENG_CHANNEL"
        android:value="${umeng_channel}" />
```

2. 在`build.gradle`配置文件中进行替换,可以在`defaultConfig`,`buildType`,`productFlavors`中配置，比如:

```groovy
defaultConfig {
    manifestPlaceholders = [ umeng_channel:"development"]
}
```

### 渠道列表进行替换

现在可以来看看如何定义渠道列表了，直接看代码：

```groovy
productFlavors {
    development {
        // 开发版
        versionCode 1
        versionName '1.0.0'
    }
    wandoujia {
        // 豌豆荚
        versionCode 2
        versionName '1.0.2'
    }
    m360 {
        // 360手机助手
        versionCode 3
        versionName '1.0.1'
    }
    ...
}
```

定义好渠道列表之后就在打包的时候进行替换，注意上面列表中标签名称即为渠道名，`productFlavor`的属性跟`defaultConfig`完全一致，可以按需修改。
替换代码如下：

```groovy
productFlavors.all { flavor ->
    flavor.manifestPlaceholders = [umeng_channel: name]
}
```

在开发的过程中可能不需要这么多渠道，只想指定的`development`渠道，怎么办呢，很简单只需要按照`buildType`的类型进行过滤即可，请看：

```groovy
variantFilter { variant ->
    def buildType = variant.buildType.name
    def flavorName = variant.getFlavors().get(0).name

    // 根据构建类型，自动过滤渠道
    if (buildType.equals('debug')) {
        if (flavorName.equals('development')) {
            variant.setIgnore(false)
        } else {
            variant.setIgnore(true)
        }
    } else {
        if (flavorName.equals('development')) {
            variant.setIgnore(true)
        } else {
            variant.setIgnore(false)
        }
    }
}

```

接下来针对打好的Release包进行重命名格式为`appName-flavorName-versionName.apk`，当然可以按需自定义，并同时放到release目录下，代码如下：

```groovy
applicationVariants.all { variant ->
    variant.outputs.each { output ->
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {
            def fileName = outputFile.name.replace(".apk", "-v${defaultConfig.versionName}.apk")
            fileName = fileName.replace("app", "app_name")
            println "[FileName]: ${fileName}"
            if (variant.buildType.name.equals('release')) {
                output.outputFile = file("${outputFile.parent}/release/${fileName}")
            } else {
                output.outputFile = file("${outputFile.parent}/${fileName}")
            }
        }
    }
}
```

好了，大功告成。但是这个过程有点长，我测试了下在我机器上打26个包大概花的时间是15分钟，所以这期间该干嘛干嘛不用老盯着它。

### 最后送上完整的build.gradle文件

```groovy
buildscript {
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.14.4'
    }
}
android {
    compileSdkVersion 21
    buildToolsVersion '21.1.1'
    defaultConfig {
        applicationId 'com.xxx.xxx'
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName '1.0.0'
        manifestPlaceholders = [umeng_channel: "development"]
    }
    sourceSets.main {
        jniLibs.srcDirs "libs"
    }
    signingConfigs {
        debug {
            storeFile file('../debug.keystore')
            storePassword 'xxx'
            keyAlias 'xxx'
            keyPassword 'xxx'
        }
        release {
            storeFile file('../debug.keystore')
            storePassword 'xxx'
            keyAlias 'xxx'
            keyPassword 'xxx'
        }
    }
    buildTypes {
        debug {
            debuggable true
            signingConfig signingConfigs.debug
        }
        release {
            debuggable false
            zipAlignEnabled true
            minifyEnabled true
            signingConfig signingConfigs.release
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }


    productFlavors {
        development {
            // 开发版
        }
        wandoujia {
            // 豌豆荚
        }
        m360 {
            // 360手机助手
        }
        xiaomi {
            // 小米商城
        }
        yingyongbao {
            // 应用宝
        }
        baidu {
            // 百度手机助手
        }
        m91zhushou {
            // 91助手
        }
        hiapk {
            // 安卓市场
        }
        anzhi {
            // 安智网
        }
        jifeng {
            // 机锋网
        }
        googleplay {
            // google play
        }
    }

    productFlavors.all { flavor ->
        flavor.manifestPlaceholders = [umeng_channel: name]
    }

    variantFilter { variant ->
        def buildType = variant.buildType.name
        def flavorName = variant.getFlavors().get(0).name

        // 根据构建类型，自动过滤渠道
        if (buildType.equals('debug')) {
            if (flavorName.equals('development')) {
                variant.setIgnore(false)
            } else {
                variant.setIgnore(true)
            }
        } else {
            if (flavorName.equals('development')) {
                variant.setIgnore(true)
            } else {
                variant.setIgnore(false)
            }
        }
    }

    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def fileName = outputFile.name.replace(".apk", "-v${defaultConfig.versionName}.apk")
                fileName = fileName.replace("app", "app_name")
                println "[FileName]: ${fileName}"
                if (variant.buildType.name.equals('release')) {
                    output.outputFile = file("${outputFile.parent}/release/${fileName}")
                } else {
                    output.outputFile = file("${outputFile.parent}/${fileName}")
                }
            }
        }
    }

    lintOptions {
        disable 'InvalidPackage'
        abortOnError false
    }

    packagingOptions {
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:support-v4:21.0.2'
    compile "com.android.support:appcompat-v7:21.0.2"
}
```

参考资料：[New build system](http://tools.android.com/tech-docs/new-build-system)
示例下载：[gradle-samples-0.14.4.zip](https://0d9321c1-a-db1c6dfe-s-sites.googlegroups.com/a/android.com/tools/tech-docs/new-build-system/gradle-samples-0.14.4.zip?attachauth=ANoY7co4TFfLH3rAwrz4yGEaSAE34iM93QStxVucYe5OAEx3Jmg1Ac_tWHpCLj4ffIQ37szRWa-TWpAbFbo34QEkN51lx60fm_0HjTJm3vS-JeNWP59sxBi36a6yx-KeBRnU2bKYxy_ELVXFWG_43DBzqFBwY28rSAtSImZiy1LhxDBYvDMvGKAUHfHVbfSZ2SHcL7NqVXEfELdiSFptDMw0NJCLf08vSMZMnomy2_uTsN-j9t7kKmb0YNjNxJbBIK1Gl3BVSLfs&attredirects=0&d=1)
