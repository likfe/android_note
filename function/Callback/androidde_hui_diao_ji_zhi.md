## 简易易懂的android回调的实现

本文讨论以下两个内容:

1. 回调函数

2. 回调机制在 Android框架 监听用户界面操作中的作用

### 一、回调函数

回调函数就是一个通过函数指针调用的函数。如果你把函数的指针(地址)作为参数传递给另一个函数，当这个指针被用为调用它所指向的函数时，我们就说这是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。

Java 中没有指针的概念，通过接口和内部类的方式实现回调的功能:

1. 定义接口 Callback ,包含回调方法 callback()

2. 在一个类Caller 中声明一个Callback接口对象 mCallback

3. 在程序中赋予 Caller对象的接口成员(mCallback) 一个内部类对象如
```java
new  Callback（）{

     callback（）{

         //函数的具体实现

     }
```
这样,在需要的时候,可用Caller对象的mCallback接口成员 调用callback()方法,完成回调.

### 二、回调机制在 Android框架 监听用户界面操作中的作用

Android事件侦听器是视图View类的接口，包含一个单独的回调方法。这些方法将在视图中注册的侦听器被用户界面操作触发时由Android框架调用。回调方法被包含在Android事件侦听器接口中：

例如,Android 的view 对象都含有一个命名为 OnClickListener 接口成员变量，用户的点击操作都会交给 OnClickListener的 OnClick() 方法进行处理。

开发者若需要对点击事件做处理，可以定义一个 OnClickListener 接口对象，赋给需要被点击的 view的接口成员变量OnClickListener，一般是用 view 的setOnClickListener() 函数来完成这一操作。

当有用户点击事件时，系统就会回调被点击view的OnClickListener接口成员的OnClick()方法。

实例(对于Android界面上Button点击事件监听的模拟):

1．定义接口
```java
public interface OnClickListener {

    public void OnClick(Button b);
}
```
2．定义Button
```java
public class Button {

  OnClickListener listener;

  public void click() {

    listener.OnClick(this);

  }

  public void setOnClickListener(OnClickListener listener) {

    this.listener = listener;

  }

}
```
3．将接口对象OnClickListener 赋给 Button的接口成员
```java
public class Activity {

  public Activity() {

  }

  public static void main(String[] args) {

    Button button = new Button();

    button.setOnClickListener(new OnClickListener(){

       @Override

       public void OnClick(Button b) {

                 System.out.println("clicked");

       }   

    });

    button.click(); //user click,System call button.click();

  }

}
```
参考资料:

>百度百科: 回调函数 http://baike.baidu.com/view/414773.html?fromTaglist

>java中回调函数的实例说明 http://www.blogjava.net/songfei/articles/126093.html

>Android事件侦听浅谈 http://developer.51cto.com/art/201001/180846.htm

>http://www.cnblogs.com/greatstar/archive/2011/03/02/1968999.html
