---
layout: post
date:   2015-03-08 22:21:49
title:   Android Data Binding
categories: Android
tags: Android Android Data Binding
---


参考：

 - [精通 Android Data Binding][1]
 - [官方教程Data Binding Guide][2]

---

1.准备

新建一个 Project，确保 Android 的 Gradle 插件版本不低于 1.5.0-alpha1：

{% highlight java %}
classpath 'com.android.tools.build:gradle:1.5.0'
{% endhighlight %}

然后修改对应模块（Module）的 build.gradle：

{% highlight java %}
dataBinding {
    enabled true
}
{% endhighlight %}

2.布局文件

使用 Data Binding 之后，xml 的布局文件就不再用于单纯地展示 UI 元素，还需要定义 UI 元素用到的变量。所以，它的根节点不再是一个 ViewGroup，而是变成了 layout，并且新增了一个节点 data。

{% highlight xml %}
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
    </data>
    <!--原先的根节点（Root Element）-->
    <LinearLayout>
    ....
    </LinearLayout>
</layout>
{% endhighlight %}

要实现 MVVM 的 ViewModel 就需要把数据（Model）与 UI（View） 进行绑定，data 节点的作用就像一个桥梁，搭建了 View 和 Model 之间的通路。

我们先在 xml 布局文件的 data 节点中声明一个 variable，这个变量会为 UI 元素提供数据（例如 TextView 的 android:text），然后在 Java 代码中把『后台』数据与这个 variable 进行绑定。

下面我们使用 Data Binding 创建一个展示用户信息的表格。

3.数据对象

添加一个 POJO 类 - User，非常简单，两个属性以及他们的 getter 和 setter。

{% highlight java %}
public class User {
    private final String firstName;
    private final String lastName;

    public User(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }
}
{% endhighlight %}

4.定义 Variable

回到布局文件，在 data 节点中声明一个 User 类型的变量 user。

{% highlight xml %}
<data>
    <variable name="user" type="com.liangfeizc.databindingsamples.basic.User" />
</data>
{% endhighlight %}

当然，data 节点也支持 import，所以上面的代码可以换一种形式来写。

{% highlight xml %}
<data>
    <import type="com.liangfeizc.databindingsamples.basic.User" />
    <variable name="user" type="User" />
</data>

{% endhighlight %}

然后我们刚才在 build.gradle 中添加的那个插件 - com.android.databinding 会根据 xml 文件的名称 Generate 一个继承自 ViewDataBinding 的类。 当然，IDE 中看不到这个文件，需要手动去 build 目录下找。

例如，这里 xml 的文件名叫 activity_basic.xml，那么生成的类就是 ActivityBasicBinding。

除了使用框架自动生成的 ActivityBasicBinding，我们也可以通过如下方式自定义类名。

{% highlight xml %}
<data class="com.example.CustomBinding">
</data>
{% endhighlight %}

5.绑定 Variable(对象)

用 DatabindingUtil.setContentView() 来替换掉 setContentView()，然后创建一个 user 对象，通过 binding.setUser(user) 与 variable 进行绑定。

{% highlight java %}
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ActivityBasicBinding binding = DataBindingUtil.setContentView(
            this, R.layout.activity_basic);
    User user = new User("fei", "Liang");
    binding.setUser(user);
}
{% endhighlight %}
6.绑定 Variable(字段)

ActivityBasicBinding 类是自动生成的，所有的 set 方法也是根据 variable 名称生成的。例如，我们定义了两个变量。

{% highlight xml %}
<data>
    <variable name="firstName" type="String" />
    <variable name="lastName" type="String" />
</data>
{% endhighlight %}

那么就会生成对应的两个 set 方法。
{% highlight java %}
setFirstName(String firstName);
setLastName(String lastName);
{% endhighlight %}

7.使用 Variable
数据与 Variable 绑定之后，xml 的 UI 元素就可以直接使用了。
{% highlight xml %}
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.lastName}" />
{% endhighlight %}

**高级用法**
---
1. 使用类方法

使用场景：需要对数据做处理再显示

首先定义一个静态方法

{% highlight java %}
public class MyStringUtils {
    public static String capitalize(final String word) {
        if (word.length() > 1) {
            return String.valueOf(word.charAt(0)).toUpperCase() + word.substring(1);
        }
        return word;
    }
}
{% endhighlight %}

然后在 xml 的 data 节点中导入：

{% highlight xml %}
<import type="com.liangfeizc.databindingsamples.utils.MyStringUtils" />
{% endhighlight %}

使用方法与 Java 语法一样：

{% highlight xml %}
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{MyStringUtils.capitalize(user.firstName)}" />
{% endhighlight %}
2. Null Coalescing 运算符

{% highlight xml %}
android:text="@{user.displayName ?? user.lastName}"
{% endhighlight %}
就等价于

{% highlight xml %}
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
{% endhighlight %}

3. Observable Binding

我们可以用任何POJO作为data binding的Model，但是直接修改POJO对象，不能直接更新UI。
Android的Data Binding模块给提供了通知机制，有3种类型，分别对应于类(Observable)，字段(ObservableField)，集合类型（Observable Collections）。
把这些observable对象绑定到View后，当observable对象更新后，UI会自动更新。

**Observable Objects用法**

我们需要把POJO继承自BaseObservable，才能获得通知UI的能力

{% highlight java %}
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getFirstName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
{% endhighlight %}

Bindable标签在编译时会自动生成BR类，但Model中的数据发生改变时，
我们在Set方法中调用notifyPropertyChanged通知UI更新。

**ObservableFields用法**

创建支持Observable的POJO类还是有点麻烦，
ObservableFields可以简化我们的POJO对象：

{% highlight java %}
private static class User extends BaseObservable {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
{% endhighlight %}

通过以下方式访问和修改字段值

{% highlight java %}
user.firstName.set("Google");
int age = user.age.get();

{% endhighlight %}

对应基础数据类型有ObservableInt、ObservableFloat、ObservableBoolean等可以使用。

**Observable Collections用法**

DataBinding中提供了一些支持通知机制的集合类型，比如ObservableArrayList，ObservableArrayMap。

ObservableArrayMap的使用跟Map一样

{% highlight java %}
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);
{% endhighlight %}

在Layout中使用ObservableArrayMap中的数据

{% highlight xml %}
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
{% endhighlight %}

  [1]: https://github.com/LyndonChin/MasteringAndroidDataBinding#%E5%AE%9A%E4%B9%89-variable
  [2]: https://developer.android.com/intl/zh-cn/tools/data-binding/guide.html
  