---
layout: post
date:   2015-03-08 22:21:49
title:   使用android studio实现多渠道打包（针对友盟），自动更新
categories: Android
tags: Android studio实现多渠道打包
---


标签（空格分隔）： Android

---

直接进入友盟官网，有详细介绍    [友盟][1]
[使用Android studio Gradle 实现友盟多渠道打包][2]

[友盟自动更新][3]

**混淆签名多渠道打包**
app的build.gradle文件
{% highlight java %}
//签名
    signingConfigs{
        release {
            storeFile file("key.jks")
            storePassword "qwe321"
            keyAlias "sdhz"
            keyPassword "qwe321"
        }
    }

    buildTypes {
        release {
            debuggable true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
{% endhighlight %}

在android studio的终端上执行命令：

gradle assembleRelease


  [1]: http://dev.umeng.com/analytics/android-doc/integration
  [2]: http://bbs.umeng.com/thread-9119-1-1.html
  [3]: http://dev.umeng.com/auto-update/android-doc/introduction