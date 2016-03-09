---
layout: post
date:   2015-03-08 22:21:49
title:   Android 音乐播放器（上）
categories: Android
tags: Android Android 音乐播放器（上）
---


标签（空格分隔）： Android

---
在实现了Activity和Service的双向通信后，我们就可以来实现音乐的播放和控制功能了

首先，在Service那到Activity的实例对象后，service想Activity发送了一条播放音乐的指令，该指令会调用service的openAudio方法，该方法首先终止其他音乐的播放（通过发送一个广播），然后再设置mediaPlayer的数据，然后调用mMediaPlayer.prepareAsync()来加载歌曲，同时我们需要设置一个加载完成的监听mMediaPlayer.setOnPreparedListener(mPreparedListener);在歌曲加载完毕后会回调onPrepare方法,一下是openAudio的方法
{% highlight java %}
public void openAudio() {
        if (audioItems == null || audioItems.isEmpty() || currentPosition == -1) {
            return ;
        }

        currentAudioItem = audioItems.get(currentPosition);

        Intent i = new Intent("com.android.music.musicservicecommand");
        i.putExtra("command", "pause");
        sendBroadcast(i);

        release();

        try {
            mMediaPlayer = new MediaPlayer();
            mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
            if(TextUtils.isEmpty(currentAudioItem.getSongId())){
                mMediaPlayer.setDataSource(AudioPlayService.this, Uri.parse
                        (currentAudioItem.getPath()));
            }else{
                mMediaPlayer.setDataSource(currentAudioItem.getPath());
            }
            mMediaPlayer.prepareAsync();
            mMediaPlayer.setOnPreparedListener(mPreparedListener);
//            mMediaPlayer.setOnCompletionListener(mCompletionListener);
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }
{% endhighlight %}
onPrepare回调：
{% highlight java %}
OnPreparedListener mPreparedListener = new OnPreparedListener() {

        @Override
        public void onPrepared(MediaPlayer mp) {
            start();
            if(mMediaPlayer.isPlaying()){
                ui.updateUI(currentAudioItem);
            }
        }
    };
{% endhighlight %}

start方法主要是开始播放音乐，并发送通知
{% highlight java %}
public void start() {
        if (mMediaPlayer != null) {
            mMediaPlayer.start();
            if(!isPlayClick)
            sendNotification();
        }
    }
{% endhighlight %}
接下来就是updateUI方法：updateUI需要做的就是：
 1. 更新播放暂停按钮的图片
 2. 设置歌手名称
 3. 设置seekBar
 4. 更新播放模式的图片
 5. 开始以一定频率更新播放进度（包括播放时间，歌词的显示位置，seekbar的进度）
 6. 如果是网络音乐则需要获取歌词
 
在初始化view的时候，我们还需要为seekbar设置拖动监听，只需要在回调方法中调用mediaPlayer的seekto方法即可

**播放暂停功能**
只需判断当前的状态，如果是播放，则调用暂停，是暂停则调用播放，然后再更新一下按钮的背景就可以了

**上一首下一首**
首先我们需要调用handler.removeCallbacksAndMessages(null)方法来暂停更新进度的操作，否则将会导致mediaPlayer的IllegalStateException，在执行上一首下一首前，我们还要获取当前用户的播放模式，我们只需要根据当前的播放模式来获取当前需要播放的数据，然后再调用openAudio方法来播放就行了
1、顺序播放
    上一首-1，下一首+1

2、随机播放
{% highlight java %}
currentPosition = random.nextInt(audioItems.size())
{% endhighlight %}
3、单曲循环不改变当前的数据位置，即不做处理

**发送通知**

[官方文档][1]
**1、创建通知**
我们需要通过NotificationCompat.Builder这个对象来指定ui的内容和动作，并至少需要设置以下3个内容：
{% highlight java %}
NotificationCompat.Builder mBuilder =
    new NotificationCompat.Builder(this)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("My notification")
    .setContentText("Hello World!");
{% endhighlight %}

我们至少需要设置一个动作：点击通知栏后跳转到activity
通过创建一个PendingIntent，然后调用mBuilder.setContentIntent(PendingIntent)来实现
{% highlight java %}
Intent resultIntent = new Intent(this, ResultActivity.class);
...
// Because clicking the notification opens a new ("special") activity, there's
// no need to create an artificial back stack.
PendingIntent resultPendingIntent =
    PendingIntent.getActivity(
    this,
    0,
    resultIntent,
    PendingIntent.FLAG_UPDATE_CURRENT
);

mBuilder.setContentIntent(resultPendingIntent);
{% endhighlight %}

**2、发送通知**

通过NotificationManager的notify方法来实现发送通知
{% highlight java %}
NotificationCompat.Builder mBuilder;
...
// Sets an ID for the notification
int mNotificationId = 001;
// Gets an instance of the NotificationManager service
NotificationManager mNotifyMgr = 
        (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
// Builds the notification and issues it.
mNotifyMgr.notify(mNotificationId, mBuilder.build());
{% endhighlight %}
我们需要指明notification ID，这个id后面还可以用来更新notification

**音乐播放器的通知栏**

在我们清除掉通知栏时，我们需要把音乐停止，这时候我们就可以为通知栏设置一个清除的回调，通过调用builder.setDeleteIntent(PendingIntent)实现

由于我们的通知栏是使用自定义的布局的，这里使用builder.setContent(RemoteViews views)方法来设置，我们看到该方法接收一个RemoteViews参数，这里也来介绍一下RemoteViews

**RemoteViews的基本使用**

这是一个可以在其他进程上展示的view，它也提供了一些修改视图的操作方法

**1、创建remoteViews**
{% highlight java %}
remoteViews = new RemoteViews(getPackageName(), R.layout.notification);
{% endhighlight %}
**2、修改文本**
{% highlight java %}
setTextViewText(int viewId, CharSequence text)
{% endhighlight %}

**3、设置点击事件**
{% highlight java %}
setOnClickPendingIntent(int viewId, PendingIntent pendingIntent)
{% endhighlight %}

**4、修改背景图片**

{% highlight java %}
setImageViewResource(int viewId, int srcId)
{% endhighlight %}

<font color="red">如果通知已经发送了，修改视图后记得重新发送一下通知，这样才会更新视图</font>
{% highlight java %}
notificationManager.notify(notificationId, notification);
{% endhighlight %}

由于篇幅问题，自定义歌词view下篇讲

  [1]: http://developer.android.com/intl/zh-cn/training/notify-user/build-notification.html#action