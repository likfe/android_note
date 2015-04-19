开发 Android APP 经常会用到自定义标题栏，而有多级页面的情况下还需要给自定义标题栏传递数据。
####本文要点：

 1. 自定义标题填充不完整
 2. 自定义标题栏返回按钮的点击事件
 
## 一、代码

这里先介绍一下流程：
 1. 创建一个标题栏布局文件 mytitlebar.xml
 2. 在style.xml中创建 mytitlestyle 主题
 3. 创建类 CustomTitleBar
 4. 在需要自定义标题栏的Activity的OnCreate方法中实例化 CustomTitleBar
 5. 在 AndroidManifest.xml 对使用了自定义标题栏的Activity定义主题

1.定义一个自定义的标题栏布局 **mytitlebar.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/re_title" xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="50dp" //定义自定义标题栏的高度
    android:background="@color/start_background"
    android:orientation="horizontal">

    <ImageButton
        android:scaleType="fitXY"
        android:layout_alignParentLeft="true"
        android:layout_centerVertical="true"
        android:layout_marginLeft="10dp"
        android:id="@+id/bt_back"
        android:layout_width="25dp"
        android:layout_height="25dp"
        android:src="@drawable/left_back"
        android:background="@color/touming"/>
    <TextView
        android:id="@+id/mytitle"
        android:layout_centerInParent="true"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:gravity="center"//使文字在整个标题栏的中间
        android:textColor="#fff"
        android:textSize="20dp" />

</RelativeLayout >
```

2.在 **style.xml** 中创建 **mytitlestyle** 主题

```
<resources>
	<!-- 自定义标题栏 parent="android:Theme" 这个属性必须写 -->
    <style name="mytitlestyle" parent="android:Theme">
	    <!-- 设置高度，和 mytitlebar.xml中保持一致 -->
        <item name="android:windowTitleSize">50dp</item>
        <!-- 设置内填充为0 使自定义标题填充整个标题栏，否则左右两边有空隙 -->
        <item name="android:padding">0dp</item>
    </style>
</resources>
```

3.创建类 CustomTitleBar

```
public class CustomTitleBar {

    private Activity mActivity;
    //不要使用 static 因为有三级页面返回时会报错

    /**
     * @param activity
     * @param title
     * @see [自定义标题栏]
     */
    public void getTitleBar(Activity activity, String title) {
        mActivity = activity;
      activity.requestWindowFeature(Window.FEATURE_CUSTOM_TITLE);
      //指定自定义标题栏的布局文件
        activity.setContentView(R.layout.mytitlebar);
        activity.getWindow().setFeatureInt(Window.FEATURE_CUSTOM_TITLE,
                R.layout.mytitlebar);
//获取自定义标题栏的TextView控件并设置内容为传递过来的字符串
        TextView textView = (TextView) activity.findViewById(R.id.mytitle);
        textView.setText(title);
        //设置返回按钮的点击事件
        ImageButton titleBackBtn = (ImageButton) activity.findViewById(R.id.bt_back);
        titleBackBtn.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
            //调用系统的返回按键的点击事件
                mActivity.onBackPressed();
            }
        });
    }
}

```

4.在需要自定义标题栏的Activity的OnCreate方法中实例化 CustomTitleBar，这里是food页面

```
public class food extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //实例化CustomTitleBar 传递相应的参数
        CustomTitleBar ct = new CustomTitleBar();
        ct.getTitleBar(this, "美食");
        setContentView(R.layout.page_food);
    }
}
```

5.在 AndroidManifest.xml 对使用了自定义标题栏的Activity定义主题

```
//省略了其余部分，android:theme="@style/mytitlestyle"这句必需写
<activity
            android:name=".food"
            android:label="@string/activity_food"
            android:theme="@style/mytitlestyle" />
```

## 二、总结
使用自定义标题栏的时候，很多人会遇到填充不满，左右两边有空隙以及返回按钮点击事件不响应的问题，这里测试和总结了最为合适的方式解决。
自定义标题栏填充不满，网上有不少解决方案，有的还比较复杂，我这里直接在定义Theme时一个属性就解决了，还比较容易理解。
自定义标题栏返回按钮点击事件不响应或出错的问题，也是测试了网上的很多代码，用onBackPressed()最为方便，也有人使用finish()，其余的OnKeyDown之类的测试未通过。

我的独立博客：[时光无罪](http://www.sinlesstime.com/)
我的CSDN博客：[他叫自己Mr.张](http://blog.csdn.net/ys743276112/)
