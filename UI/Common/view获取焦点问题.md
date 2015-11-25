## view获取焦点问题

如果`listitem`里面包括`button`或者`checkbox`等控件，默认情况下`listitem`会失去焦点，导致无法响应`item`的事件。

### 方法一
最常用的解决办法是在`listitem`的布局文件中设置`descendantFocusability`属性。
item的布局文件：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:paddingTop="10dp"
  android:paddingBottom="10dp"
  android:paddingLeft="5dp"
  android:paddingRight="5dp"
  android:descendantFocusability="blocksDescendants"><!--添加这个属性-->
  <CheckBox
   android:id="@+id/history_item_checkbt"
   android:layout_height="30dp"
   android:layout_width="wrap_content"
   android:layout_centerVertical="true"
   android:layout_alignParentLeft="true"
   android:checked="false"
   >
  </CheckBox>

  <ImageView
   android:id="@+id/history_item_image"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:layout_centerVertical="true"
   android:layout_toRightOf="@id/history_item_checkbt"
   android:background="@drawable/item_icon">
  </ImageView>

  
  <Button
   android:id="@+id/history_item_edit_bt"
   android:layout_alignParentRight="true"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:layout_centerVertical="true"
   android:text="编辑"
   android:textColor="#ffffff"
   android:textSize="14sp"
   android:background="@drawable/button_bg">
  </Button>

  <TextView
   android:id="@+id/history_item_time_tv"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:layout_centerVertical="true"
   android:textColor="#565C5D"
   android:textSize="14sp"
   android:text="10-01 10:20"
   android:layout_marginRight="5dp"
   android:layout_toLeftOf="@id/history_item_edit_bt">
  </TextView>

  <TextView
   android:id="@+id/history_item_title_tv"
   android:layout_height="wrap_content"
   android:layout_width="fill_parent"
   android:layout_centerVertical="true"
   android:textColor="#565C5D"
   android:textSize="14sp"
   android:text="xxxxxxxxXXXXXXXXXXXXXXXX"
   android:ellipsize="end"
   android:maxLines="1"
   android:layout_toRightOf="@id/history_item_image"
   android:layout_toLeftOf="@id/history_item_time_tv"
   android:layout_marginLeft="3dp">
  </TextView>
</RelativeLayout>
```

> **android:descendantFocusability**

> Defines the relationship between the ViewGroup and its descendants when looking for a View to take focus.

> Must be one of the following constant values.
![](http://img.my.csdn.net/uploads/201210/17/1350460358_5684.jpg)

该属性是当一个为view获取焦点时，定义viewGroup和其子控件两者之间的关系。属性的值有三种：
* beforeDescendants：viewgroup会优先其子类控件而获取到焦点
* afterDescendants：viewgroup只有当其子类控件不需要获取焦点时才获取焦点
* blocksDescendants：viewgroup会覆盖子类控件而直接获得焦点

我们使用的是第三个。

### 方法二
在子控件（Button等）里设置`android:focusable="false"`，使得其无法获取焦点，从而不会干扰`item`的点击事件。
