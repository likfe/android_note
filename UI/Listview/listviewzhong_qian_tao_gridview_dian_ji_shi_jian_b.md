## ListView中嵌套Gridview点击事件不响应

首先在Listview的item的XML文件的最外层加入
`android:descendantFocusability="blocksDescendants"`

然后在adapter的java文件中获取gridview

设置
```
holder.imgGrid.setClickable(false);
holder.imgGrid.setPressed(false);
holder.imgGrid.setEnables(false);
```

具体案例如下:

