---
layout: post
date:   2015-03-08 22:21:49
title:   更新gradle plugin、gradle version
categories: Android
tags: Android 更新gradle plugin、gradle version
---



标签（空格分隔）： Android

---
**gradle plugin**

在project的build.gradle文件中配置，只需更改后面的版本号就可以，as会自动联网下载。

这里可以查看最新的gradle plugin版本：[http://tools.android.com/tech-docs/new-build-system][1]

{% highlight java %}
dependencies {
        classpath 'com.android.tools.build:gradle:2.0.0-alpha5'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
{% endhighlight %}

**gradle version**

1、下载gradle version

翻墙：

{% highlight java %}
https://downloads.gradle.org/distributions/gradle-2.10-all.zip
下载其它版本把“2.10”替换成你所需要的版本号就ok啦
下载地址：
https://downloads.gradle.org/distributions/gradle-2.10-all.zip
{% endhighlight %}

可以在AndroidDevTools中下载，不需翻墙
[AndroidDevTools][2]

下载完成解压到任意路径。建议放到android studio目录下。

2、 studio中配置
Settings > Builds,Execution,Deployment > Build Tools > Gradle >Gradle home path

完成。

3、修改项目的gradle版本
项目下>gradle>wrapper>gradle-wrapper.properties

{% highlight java %}
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip
{% endhighlight %}





  [1]: http://tools.android.com/tech-docs/new-build-system
  [2]: http://www.androiddevtools.cn/