title: android项目开发：UI设计
author: 归零幻想
publishDate: 2020-12-21
editDate: 2020-12-21
tags: [android, 第一行代码, Kotlin]

<!--config-->

仍然是《第一行代码》的学习笔记，这里记录的东西相对少一点，UI上的东西还是更多在实际项目中感受到。比如我想没有必要写TextView的介绍吧。

## 控件的使用方法

`dp`是一种屏幕密度无关的尺寸单位，可以保证在不同分辨率的手机上显示效果尽可能一致。 

`match_parent`表示让当前控件大小和父布局的大小一致。

`wrap_content`表示让当前控件的大小能正好包裹里面的内容。

`android:gravity`指定控件内的内容对齐方式，有`top`、`bottom`、`start`、`end`、`center`等可选，可以用`|`指定多个值。比如`center`等价于`center_vertical|center_horizonal`。

## 基本布局
### LinearLayout

线性布局，通过`android:orientation`指定方向。

有一个重要属性：`android:layout_weight`，它将控件已经占用的空间减掉后按照比重分给各个控件。一般我们直接指定`android:layout_width`为0dp，而给它指定一个比重，这样控件的尺寸将占满剩余空间。

<!--summary-->

### RelativeLayout

相对布局，复杂，但有迹可循。

```

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:text="Button1" />

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_alignParentRight="true"
        android:text="Button2" />

    <Button
        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Button3" />

    <Button
        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentBottom="true"
        android:text="Button4" />

    <Button
        android:id="@+id/button5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentBottom="true"
        android:text="Button5" />
</RelativeLayout>
```

以上这段布局文件描述的就是一个相对布局，共有5个按钮，分别在父布局的左上、右上、中间、左下、右下位置。

相对布局不仅可以相对于父布局，也可以相对于控件。以下描述了button3在中间，左上button1，右上button2，左下button4，右下button5的布局场景。

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Button3" />

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/button3"
        android:layout_toLeftOf="@id/button3"
        android:text="Button1" />

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/button3"
        android:layout_toRightOf="@id/button3"
        android:text="Button2" />

    <Button
        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/button3"
        android:layout_toLeftOf="@id/button3"
        android:text="Button4" />

    <Button
        android:id="@+id/button5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/button3"
        android:layout_toRightOf="@id/button3"
        android:text="Button5" />
</RelativeLayout>
```

### FrameLayout

帧布局，其内部组件将按顺序层叠到左上角。

> FrameLayout一直有foreground属性，而View在后续版本才支持，在一些需要做半透明效果的地方可以用FrameLayout的属性实现，减少控件层叠层数，保证兼容性。

## 自定义控件

安卓中所有的控件都是直接或间接继承自View的，而所有的布局都是继承自ViewGroup的。

View是安卓中最基本的一种UI组件，可以在屏幕上绘制一块矩形区域，并响应这块区域的各种事件。

ViewGroup是一种特殊的View，可以包含很多子View和子ViewGroup。

## RecyclerView

基本用法：

1. 导入`androidx.recyclerview:recyclerview`，
2. 定义类继承RecyclerView.Adapter和RecyclerView.ViewHolder。
3. 实现。

使用DiffUtil可以简化数据更新的写法。

参考[我的一个demo项目](https://github.com/zerofancy/ipviewer)

常用布局管理器LinearLayoutManager（线性），此外还有GridLayoutManager（网格）和StaggeredGridLayoutManager（瀑布流）。

> LinearLayoutManager和PagerSnapHelper配合可以实现类似抖音的刷视频的效果。
> 在引入`kotlin-android-extensions`之后可以比较方便地在`onBindViewHolder`方法中给具体的view赋值。
