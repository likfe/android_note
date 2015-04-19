1.有人想通过下面代码来实现手动调用返回键，很可惜会出现空指针异常。

```
    this.onKeyDown(KeyEvent.KEYCODE_BACK, null);
```
我们可以通过调用
```
onBackPressed();

注解: 封装时使用动态方法定义类成员
调用也使用动态方法 否则 主页》二级页面》三级页面 返回到二级页面 再返回就出错
```
来实现实现调用返回键。
2.如果想要按下返回键时附加执行一些代码，可以写在这里
```
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
   if (keyCode == KeyEvent.KEYCODE_BACK && event.getRepeatCount() == 0) {
       // Do something.
       return true;
   }
   return super.onKeyDown(keyCode, event);
}
```