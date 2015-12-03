# tabhost英文字母自动变大写问题

android tabhost为什么有的设置方式会变成英文字母大写，但有的时候不会？

如果是4.0以上出现字母大写，可以执行以下code：
```java
TabWidget tabWidget = tHost.getTabWidget();
		for (int i = 0; i < tabWidget.getChildCount(); i++) {
			TextView tv = (TextView) tabWidget.getChildAt(i).findViewById(
					android.R.id.title);
			tv.setTransformationMethod(null);//不设置为大写
		}
```