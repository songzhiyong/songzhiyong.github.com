---
layout: post
title: "[转] Layout Inflation不能这么用"
date: 2014-07-05 15:27
comments: true
categories: android
---


转自：[http://blog.jobbole.com/72156/](http://blog.jobbole.com/72156/)

Layout inflation在Android上下文环境下转换XML文件成View结构对象的时候需要用到。

LayoutInflater这个对象在Android的SDK中很常见，但是你绝对没想到竟然能够找到一个使用误区。说不定你的App里就是这么用的！如果你在写APP的时候像如下代码一样使用LayoutInflater的话：

```java
inflater.inflate(R.layout.my_layout, null);
```

请你继续读完这篇文章，稍后我会解释为什么这样做不对。

###认识LayoutInflater

首先看一下LayoutInflater的工作原理，有两个重载的版本可以使用：

inflate(int resource, ViewGroup root) 和 inflate(int resource, ViewGroup root, boolean attachToRoot)

第一个参数指出要载入的布局文件资源，第二个参数指出视图结构中载入的布局将要放入的根视图。如果有第三个参数，那么它用来决定是否把载入后的视图绑定到给出的根视图中。

最后两个参数可能会导致一些问题。如果使用两个参数的版本，Layoutinflater会自动尝试把载入的视图绑定到给定的根视图对象中。但是，如果你传递null，系统就不会尝试绑定操作了，否则应用程序就崩溃了。

很多开发者会这样做，认为传递null作为根视图就可以禁用绑定操作了。很多时候很多开发者甚至不知道还有三个参数的Layoutinflater版本的存在，如果这么做的话，也会同时禁用了根视图的一个很重要的函数……但是之前我没有研究过。

###框架中的示例

现在我们来仔细看看Android框架关于动态载入布局的场景。

Adapter是最常用的场景，我们经常需要使用LayoutInflater来自定义ListView（通过重写getView()方法），具体的方法签名是这样的：

```java
getView(int position, View convertView, ViewGroup parent)
```

Fragment也会用到inflation操作，通过onCreateView()方法创建view的时候会用到。这个方法的签名是这样的：

```java
onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)
```

不知你有没有注意到这一点，每次Framework需要你去载入一个布局文件时，都会传入一个ViewGroup参数（最后需要绑定到的根视图），如果Layoutinflater设为自动绑定到根视图的话，会抛出一个异常。

所以你想想看，如果我做绑定操作的话，为什么要给你一个ViewGroup参数呢？事实证明父视图在这个inflation操作过程中是很重要的，它会计算被载入的XML在根元素中的LayoutParams，如果传入null话，就等于是告诉框架“我不知道载入的View要放到哪个父视图中”。

问题在于，android:layout_xxx属性会在父视图对象中被重新计算，结果就是所有你定义的LayoutParams都会被忽略掉（因为没有已知的父视图对象）。然后你就纳闷“为什么框架会忽略掉我自己定义的布局属性呢？还是去StackOverFlow上看看，提一个bug吧”。

如果没有设置LayoutParams，那么最终ViewGroup也会给你生成一个默认的属性，幸运的话（很多时候），这些默认的设置正好和你在XML文件中定义的一样……所以你就察觉不到其实已经出现问题了。

###应用案例

你敢说你没有在应用中碰到过这样的场景吗？看看下面的代码，为Listview简单地载入一个布局文件：

R.layout.item_row

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:gravity="center_vertical"
    android:orientation="horizontal">
    <TextView
        android:id="@+id/text1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingRight="15dp"
        android:text="Text1" />
    <TextView
        android:id="@+id/text2"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Text2" />
</LinearLayout>
```

这里我们想把高度设置为固定高度，上面把它设为当前主题下的推荐高度……看似很合理。

但是，当我们这样载入布局文件的时候，就不对了：

```java
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {
        convertView = inflate(R.layout.item_row, null);
    }
  
    return convertView;
}
```

然后结果就变成这样了：

![](/images/2014/07/image-1.png "")<br/>

为什么设定的固定高度不起作用？这是因为你没有把所有子View的高都设为固定高度，只需要把根视图的高设置成wrap_content就可以了。不需要知道为什么会这样（你可以吐槽一下Google为什么这么处理！）。

而如果这样载入布局的话就没有问题：

```java
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {
        convertView = inflate(R.layout.item_row, parent, false);
    }
  
    return convertView;
}
```

这样我们就得到了想要的结果：

![](/images/2014/07/image-2.png "")<br/>

###任何规则都有例外

当然，也有需要在载入布局的时候指定null作为父布局对象，但这种情况非常少。一个典型的例子就是为AlertDialog中载入一个自定义布局。看看下面的例子，使用和上面一样的XML布局文件来作为对话框的布局：

```java
AlertDialog.Builder builder = new AlertDialog.Builder(context);
View content = LayoutInflater.from(context).inflate(R.layout.item_row, null);
  
builder.setTitle("My Dialog");
builder.setView(content);
builder.setPositiveButton("OK", null);
builder.show();
```

这里的问题就是，AlertDialog.Builder支持自定义布局，但是setView()方法不提供带有布局文件作为参数的版本，所以只能先手动载入XML布局文件。由于最终会进入到对话框里面，不会接触到根布局（事实上这时候还没有根布局），所以我们也操作不了布局文件的最终父视图对象，当然也就不能用于载入使用了。事实证明，这些都是无关紧要的，因为AlertDialog会擦除布局上的所有Layoutparams然后替换为match_parent。

所以，下次使用inflate()函数时，如果还想输入null应该停下来想一想“我真的不知道它该放到哪里吗？”

最后，你应该想想两个参数的inflate()版本作为一个便捷的使用方式，可以忽略第三个参数（默认为true），但是不要想着为了方便而传递一个null却忽略了第三个参数会默认是false。

