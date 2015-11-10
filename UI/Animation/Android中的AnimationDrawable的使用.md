## Android中的AnimationDrawable的使用

首先可以先定义一个逐帧播放的xml：
```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false" >

    <item
        android:drawable="@drawable/on_001"
        android:duration="100"/>

    <item
        android:drawable="@drawable/on_002"
        android:duration="100"/>

    <item
        android:drawable="@drawable/on_003"
        android:duration="100"/>

    <item
        android:drawable="@drawable/on_004"
        android:duration="100"/>

    <item
        android:drawable="@drawable/on_005"
        android:duration="100"/>

    <item
        android:drawable="@drawable/on_006"
        android:duration="100"/>
</animation-list>
```
**注解：** 
1. `android:oneshot`表示是否播放一次，值为`true`则表示播放一次动画，值为`false`，则表示动画一直循环播放。
2. 每一个`item`表示动画的一个阶段
3. `android:drawable`指定播放的文件，`android:duration`指定播放的时间，单位为毫秒。

然后在代码中定义出AnimationDrawable对象，并设置到view的background上，然后设置开始播放就可以了：
```java
AnimationDrawable ad = (AnimationDrawable) getResources().getDrawable(
                 R.drawable.bootanimation);
         mView.setBackgroundDrawable(ad);
         ad.start();
```

