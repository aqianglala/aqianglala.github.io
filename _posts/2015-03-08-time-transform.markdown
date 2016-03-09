---
layout: post
date:   2015-03-08 22:21:49
title:   时间格式的转换
categories: Android
tags: Android 时间格式的转换
---



---

{% highlight java %}
/**
     * 2015-11-25T08:40:42.166Z------->Date
     * @param date
     * @return
     */
    public static Date getDate(String date) {
        final String year = date.substring(0, 4);
        final String month = date.substring(5, 7);
        final String day = date.substring(8, 10);
        final String hour = date.substring(11, 13);
        final String minute = date.substring(14, 16);
        final String second = date.substring(17, 19);
        final int millisecond = Integer.valueOf(date.substring(20, 23));
        Calendar result =
                new GregorianCalendar(Integer.valueOf(year),
                        Integer.valueOf(month) - 1, Integer.valueOf(day),
                        Integer.valueOf(hour), Integer.valueOf(minute),
                        Integer.valueOf(second));
        result.set(Calendar.MILLISECOND, millisecond);
        result.setTimeZone(TimeZone.getTimeZone("Etc/UTC"));
        Date datess = new Date(result.getTimeInMillis());
        return datess;
    }

    /**
     * 2015-11-25T08:40:42.166Z------->yyyy'年'MM'月'dd'日'HH:mm
     * 有时区计算，转换为UTC时间格式
     * @param date
     *            the String to read the date from
     * @return a calendar representing the date found in the string
     */
    public static String getStringToCal(String date) {
        final String year = date.substring(0, 4);
        final String month = date.substring(5, 7);
        final String day = date.substring(8, 10);
        final String hour = date.substring(11, 13);
        final String minute = date.substring(14, 16);
        final String second = date.substring(17, 19);
        final int millisecond = Integer.valueOf(date.substring(20, 23));
        Calendar result =
                new GregorianCalendar(Integer.valueOf(year),
                        Integer.valueOf(month) - 1, Integer.valueOf(day),
                        Integer.valueOf(hour), Integer.valueOf(minute),
                        Integer.valueOf(second));
        result.set(Calendar.MILLISECOND, millisecond);
        result.setTimeZone(TimeZone.getTimeZone("Etc/UTC"));
        Date datess = new Date(result.getTimeInMillis());
        SimpleDateFormat format = new SimpleDateFormat("yyyy'年'MM'月'dd'日'HH:mm");
        return format.format(datess);
    }

/**
     * 将当前时间转成gmt格式，即包含TZ
     * @param date
     * @return
     */
    public static String getGMTTime(Date date){
        SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.sss'Z'");
        format.setTimeZone(TimeZone.getTimeZone("GMT"));
        System.out.println(format.format(date));
        return format.format(date);
    }
{% endhighlight %}




