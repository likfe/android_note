##  Android Service被关闭后自动重启,解决被异常kill服务
Android开发的过程中，每次调用`startService(Intent)`的时候，都会调用该`Service`对象的`onStartCommand(Intent,int,int)`方法，然后在`onStartCommand`方法中做一些处理。然后我们注意到这个函数有一个`int`的返回值，这篇文章就是简单地讲讲`int`返回值的作用。
从Android官方文档中，我们知道onStartCommand有4种返回值：
 
1. **START_STICKY：**如果`service`进程被kill掉，保留`service`的状态为开始状态，但不保留递送的`intent`对象。随后系统会尝试重新创建`service`，由于服务状态为开始状态，所以创建服务后一定会调用`onStartCommand(Intent,int,int)`方法。如果在此期间没有任何启动命令被传递到`service`，那么参数`Intent`将为`null`。
 
2. **START_NOT_STICKY：**“非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统**不会自动重启**该服务。
 
3. **START_REDELIVER_INTENT**：重传Intent。使用这个返回值时，如果在执行完`onStartCommand`后，服务被异常`kill`掉，系统会**自动重启**该服务，并将Intent的值传入。

4. **START_STICKY_COMPATIBILITY**：`START_STICKY`的兼容版本，但不保证服务被`kill`后一定能重启。

>参考：http://www.eoeandroid.com/thread-169411-1-1.html

