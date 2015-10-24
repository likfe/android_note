## 自定义ListView按下颜色

最近手上有个小项目，项目中用到了ListView，但凡用过ListView的童鞋都知道，在像Android2.3这样使用广泛的安卓版本上，当它被点按的时候会产生黄色的高亮效果，相当恶心，相当难看，而且在不同版本的安卓上效果还不一样，无法做到软件风格的统一。

为了使软件在不同安卓平台上运行时尽量保持统一的风格，我试图修改颜色，网络上有人说使用android:listSelector这个属性，可是如果单单使用这个属性去修改的话ListView会有一种被选中的效果，体验不是很好，于是我舍近求远，折腾出了以下这种比较**的做法

首先在Listview的布局中添加以下属性
```xml
android:listSelector="@android:color/transparent"
```
这时点击Listview上面的项就没有了按下的效果，因为我使用透明替代了
然后在写Item的布局时对其背景进行自定义
```xml
android:background="@drawable/bg_item"
```
bg_item.xml如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
 
    <item android:drawable="@android:color/white" android:state_pressed="false"/>
    <item android:drawable="@color/holo_blue_light" android:state_pressed="true"/>
 
</selector>
```
这样的话就可以自定义ListView的按下效果，再也不是讨厌的黄色了

> http://my.oschina.net/nylqd/blog/139185

### 取消点击效果
方法一，在控件被初始化的时候设置
```java
gridView.setSelector(new ColorDrawable(Color.TRANSPARENT));
listView.setSelector(new ColorDrawable(Color.TRANSPARENT))；
```
方法二，在布局文件中设置listSelector属性
```xml
<GridView
        android:listSelector="@android:color/transparent"
        android:numColumns="auto_fit"
        android:columnWidth="50dp"
        android:stretchMode="spacingWidth"
        android:layout_weight="1.0"
        android:layout_height="0dip"
        android:layout_width="match_parent"/>
 
<ListView
        android:listSelector="@android:color/transparent"
        android:layout_height="match_parent"
        android:layout_width="match_parent"/>
```

当然也可以定制化自己想要的效果。 
推荐使用方法二，解耦逻辑代码与布局文件。