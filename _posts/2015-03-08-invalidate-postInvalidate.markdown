---
layout: post
date:   2015-03-08 22:21:49
title:   invalidate()和postInvalidate() 的区别及使用
categories: Android
tags: Android invalidate()和postInvalidate() 的区别及使用
---


---

直接看官方解释：

**invalidate**

必须在UI线程中调用，view的方法会全部重新调用
{% highlight java %}
/**
     * Invalidate the whole view. If the view is visible,
     * {@link #onDraw(android.graphics.Canvas)} will be called at some point in
     * the future.
     * <p>
     * This must be called from a UI thread. To call from a non-UI thread, call
     * {@link #postInvalidate()}.
     */
    public void invalidate() {
        invalidate(true);
    }
{% endhighlight %}

**postInvalidate**

在子线程中调用，view的方法同样会全部重新调用
{% highlight java %}
/**
     * <p>Cause an invalidate to happen on a subsequent cycle through the event loop.
     * Use this to invalidate the View from a non-UI thread.</p>
     *
     * <p>This method can be invoked from outside of the UI thread
     * only when this View is attached to a window.</p>
     *
     * @see #invalidate()
     * @see #postInvalidateDelayed(long)
     */
    public void postInvalidate() {
        postInvalidateDelayed(0);
    }
{% endhighlight %}




