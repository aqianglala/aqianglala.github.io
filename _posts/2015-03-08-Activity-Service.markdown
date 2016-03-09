---
layout: post
date:   2015-03-08 22:21:49
categories:   Android
tags: Android Activity和service交互
---

标签（空格分隔）： Android

---

> 在播放音乐时，要实现在activity被销毁时也能进行对音乐的控制的需求，因此我们可以选择使用Service。先来回顾一下service，可参考以下文章

[ Android Service完全解析，关于服务你所需知道的一切(上)][1]

学习service主要学习一下几点：

 - 启动service的两种方式（startService和bindService）
 - 停止service的两种方式（stopService和unbindService）
 - 两种启动方式所执行的生命周期方法的不同
 - 两种停止方式所执行的生命周期方法的不同
 - Service和Activity通信（单向：activity调用service的方法，通过bindservice实现）
 - Service和Activity通信（双向：通过messenger传递各自的handler实现）
 - 前台service
 - AIDL实现某个Service与多个应用程序组件之间进行跨进程通信
 
这里主要讲一下Service和Activity的双向通信

**Service和Activity的双向通信**
核心思想：让对方获取到自己的实例对象，利用这个实例对象就可以调用其方法实现通信
问题是，如何让对方获取到自己的实例对象呢？我们知道，通过bindService我们可以获取到service的对象，问题在于如何让service获取到Acitivity对象，这里我们假设，如果能让双方都获取到各自的handler对象，那么我们就可以向这个handler发送自己的实例对象了，那现在问题就简化为：如何让对方获取到各自的handler对象！
其实官方已经提供了一个类用来充当信使了，我们只需要让双方都获取到各自的信使就ok了，现在我们就来介绍一下这个信使：Messenger

**Messenger**
[android Messenger 跨进程通信][2]

官方文档解释：它引用了一个Handler对象，以便others能够向它发送消息(使用mMessenger.send(Message msg)方法)。该类允许跨进程间基于Message的通信(即两个进程间可以通过Message进行通信)，在服务端使用Handler创建一个Messenger，客户端持有这个Messenger就可以与服务端通信了。

**1、服务端创建一个信使对象**
{% highlight java %}
mMessenger = new Messenger(mHandler)
{% endhighlight %}
**２、客户端使用bindlerService请求连接远程**

**３、远程onBind方法返回一个bindler**
{% highlight java %}
return mMessenger.getBinder();
{% endhighlight %}
**4、客户端使用远程返回的bindler得到一个信使（即得到远程信使）**
{% highlight java %}
public void onServiceConnected(ComponentName name, IBinder service) {
	rMessenger = new Messenger(service);
　　　　　　......
}
{% endhighlight %}
**５、客户端可以使用这个远程信使对象向远程发送消息：rMessenger.send(msg)**

**6、实现双向传递**
经过这５个步骤貌似只有客户端向服务端发送消息，这样的消息传递是单向的，那么如何实现双向传递呢？
首先需要在第５步稍加修改，在send(msg)前通过msm.replyTo = mMessenger将自己的信使设置到消息中，这样服务端接收到消息时同时也得到了客户端的信使对象了，然后服务端可以通过

{% highlight java %}
//得到客户端的信使对象，并向它发送消息
cMessenger = msg.replyTo;
cMessenger.send(message);
{% endhighlight %}

双向通信宣告完成。

  [1]: http://blog.csdn.net/guolin_blog/article/details/11952435
  [2]: http://blog.csdn.net/cs_lht/article/details/6698468