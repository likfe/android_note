## 如何让Service不被杀死
1. 在service中重写下面的方法，这个方法有三个返回值， START_STICKY是service被kill掉后自动重写创建
```
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
        return START_STICKY;
    }

  @Override
public int onStartCommand(Intent intent, int flags, int startId) {
                Log.v("TrafficService","startCommand");
                
                flags =  START_STICKY;
                return super.onStartCommand(intent, flags, startId);
                //return  START_REDELIVER_INTENT;
        }
```
2. 在Service的onDestroy()中重启Service.
```
public void onDestroy() {   
        Intent localIntent = new Intent();
        localIntent.setClass(this, MyService.class);  //销毁时重新启动Service
        this.startService(localIntent);
    }
```
用qq管家杀掉进程的时候，调用的是系统自带的强制kill功能（即settings里的），在kill时，会将应用的整个进程停掉，当然包括service在内，如果在running里将service强制kill掉，显示进程还在。不管是kill整个进程还是只kill掉进应用的 service，都不会重新启动service。不知道你是怎么怎么实现重启的，实在是不解。。。
ps：在eclipse中，用stop按钮kill掉进程的时候，倒是会重启service
KILL问题：
1.settings 中stop service
onDestroy方法中，调用startService进行Service的重启。
2.settings中force stop 应用
捕捉系统进行广播（action为android.intent.action.PACKAGE_RESTARTED）
3.借助第三方应用kill掉running task
提升service的优先级

3. service开机启动
今天我们主要来探讨android怎么让一个service开机自动启动功能的实现。Android手机在启动的过程中会触发一个Standard Broadcast Action，名字叫android.intent.action.BOOT_COMPLETED(记得只会触发一次呀),在这里我们可以通过构建一个广播接收者来接收这个这个action.下面我就来简单写以下实现的步骤：  
第一步：首先创建一个广播接收者,重构其抽象方法 onReceive(Context context, Intent intent)，在其中启动你想要启动的Service或app。
```
import android.content.BroadcastReceiver;  
    import android.content.Context;  
    import android.content.Intent;  
    import android.util.Log;  
      
    public class BootBroadcastReceiver extends BroadcastReceiver {  
        //重写onReceive方法  
        @Override  
        public void onReceive(Context context, Intent intent) {  
            //后边的XXX.class就是要启动的服务  
            Intent service = new Intent(context,XXXclass);  
            context.startService(service);  
            Log.v("TAG", "开机自动服务自动启动.....");  
           //启动应用，参数为需要自动启动的应用的包名
    Intent intent = getPackageManager().getLaunchIntentForPackage(packageName);
    context.startActivity(intent );        
        }  
      
    }
```
第二步：配置xml文件，在receiver接收这种添加intent-filter配置 
```xml
<receiver android:name="BootBroadcastReceiver">  
    <intent-filter>  
        <action android:name="android.intent.action.BOOT_COMPLETED"></action>  
        <category android:name="android.intent.category.LAUNCHER" />  
    </intent-filter>  
</receiver>
```
第三步：添加权限 `<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" /> ` 
4. 如何实现一个不会被杀死的进程
看Android的文档知道，当进程长期不活动，或系统需要资源时，会自动清理门户，杀死一些Service，和不可见的Activity等所在的进程。
但是如果某个进程不想被杀死（如数据缓存进程，或状态监控进程，或远程服务进程），应该怎么做，才能使进程不被杀死。

add android:persistent="true" into the <application> section in your AndroidManifest.xml

切记，这个不可滥用，系统中用这个的service，app一多，整个系统就完蛋了。
目前系统中有phone等非常有限的，必须一直活着的应用在试用。

------------------------------------------------
提升service优先级的方法

Android 系统对于内存管理有自己的一套方法，为了保障系统有序稳定的运信，系统内部会自动分配，控制程序的内存使用。当系统觉得当前的资源非常有限的时候，为了保 证一些优先级高的程序能运行，就会杀掉一些他认为不重要的程序或者服务来释放内存。这样就能保证真正对用户有用的程序仍然再运行。如果你的 Service 碰上了这种情况，多半会先被杀掉。但如果你增加 Service 的优先级就能让他多留一会，我们可以用 setForeground(true) 来设置 Service 的优先级。 

　　为什么是 foreground ? 默认启动的 Service 是被标记为 background，当前运行的 Activity 一般被标记为 foreground，也就是说你给 Service 设置了 foreground 那么他就和正在运行的 Activity 类似优先级得到了一定的提高。当让这并不能保证你得 Service 永远不被杀掉，只是提高了他的优先级。 
　　从Android 1.5开始，一个已启动的service可以调用startForeground(int, Notification)将service置为foreground状态，调用stopForeground(boolean)将service置为 background状态。 
　　我们会在调用startForeground(int, Notification)传入参数notification，它会在状态栏里显示正在进行的foreground service。background service不会在状态栏里显示。 
　　在Android 1.0中，将一个service置为foreground状态： 
　　setForeground(true); 
　　mNM.notify(id, notification); 
　　将一个service置为background状态： 
　　mNM.cancel(id); 
　　setForeground(false); 
　　对比看出，在1.0 API中调用setForeground(boolean)只是简单的改变service的状态，用户不会有任何觉察。新API中强制将 notification和改变service状态的动作绑定起来，foreground service会在状态栏显示，而background service不会。 
　　Remote service controller & binding 
跨进程调用Service。暂时不研究。 

///

如何防止Android应用中的Service被系统回收?         很多朋友都在问，如何防止Android应用中的Service被系统回收？下面简单解答一下。

对于Service被系统回收，一般做法是通过提高优先级可以解决，在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时实用于广播，推荐大家如果你的应用很重要，可以考虑通过系统常用intent action来触发。 
