## MediaPlayer播放网络音频
以前曾经地介绍过[MediaPlayer的基本用法](http://blog.csdn.net/hellogv/archive/2010/10/30/5975864.aspx)，这里就深入地讲解MediaPlayer的在线播放功能。本文主要实现MediaPlayer在线播放音频的功能，由于在线视频播放比在线音频播放复杂，因此先介绍在线音频播放的实现，这样可以帮助大家逐步深入了解MediaPlayer的在线播放功能。先来看看本文程序运行的结果：


main.xml的源码如下：
```xml
<?xml version="1.0" encoding="utf-8"?>  
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_height="fill_parent" android:layout_width="fill_parent">  
    <LinearLayout android:layout_height="wrap_content"  
        android:layout_width="fill_parent" android:orientation="vertical"  
        android:layout_gravity="top">  
        <LinearLayout android:orientation="horizontal"  
            android:layout_gravity="center_horizontal" android:layout_marginTop="4.0dip"  
            android:layout_height="wrap_content" android:layout_width="wrap_content">  
            <Button android:layout_width="wrap_content"  
                android:layout_height="wrap_content" android:id="@+id/btnPlayUrl"  
                android:text="播放网络音频"></Button>  
            <Button android:layout_height="wrap_content" android:id="@+id/btnPause"  
                android:text="暂停" android:layout_width="80dip"></Button>  
            <Button android:layout_height="wrap_content"  
                android:layout_width="80dip" android:text="停止" android:id="@+id/btnStop"></Button>  
        </LinearLayout>  
        <LinearLayout android:orientation="horizontal"  
            android:layout_width="fill_parent" android:layout_height="wrap_content"  
            android:layout_marginBottom="20dip">  
            <SeekBar android:paddingRight="10dip" android:layout_gravity="center_vertical"  
                android:paddingLeft="10dip" android:layout_weight="1.0"  
                android:layout_height="wrap_content" android:layout_width="wrap_content"  
                android:id="@+id/skbProgress" android:max="100"></SeekBar>  
        </LinearLayout>  
    </LinearLayout>  
</FrameLayout>  
```
`Player.java`是本文的核心，`Player.java`实现了“进度条更新”、“数据缓冲”等功能，虽然不是很复杂的功能，但却是非常有用的功能。`Player.java`源码如下：

```java
package com.musicplayer;  
  
import java.io.IOException;  
import java.util.Timer;  
import java.util.TimerTask;  
import android.media.AudioManager;  
import android.media.MediaPlayer;  
import android.media.MediaPlayer.OnBufferingUpdateListener;  
import android.media.MediaPlayer.OnCompletionListener;  
import android.os.Handler;  
import android.os.Message;  
import android.util.Log;  
import android.widget.SeekBar;  
  
public class Player implements OnBufferingUpdateListener,  
        OnCompletionListener, MediaPlayer.OnPreparedListener{  
    public MediaPlayer mediaPlayer;  
    private SeekBar skbProgress;  
    private Timer mTimer=new Timer();  
    public Player(SeekBar skbProgress)  
    {  
        this.skbProgress=skbProgress;  
          
        try {  
            mediaPlayer = new MediaPlayer();  
            mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);  
            mediaPlayer.setOnBufferingUpdateListener(this);  
            mediaPlayer.setOnPreparedListener(this);  
        } catch (Exception e) {  
            Log.e("mediaPlayer", "error", e);  
        }  
          
        mTimer.schedule(mTimerTask, 0, 1000);  
    }  
      
    /******************************************************* 
     * 通过定时器和Handler来更新进度条 
     ******************************************************/  
    TimerTask mTimerTask = new TimerTask() {  
        @Override  
        public void run() {  
            if(mediaPlayer==null)  
                return;  
            if (mediaPlayer.isPlaying() && skbProgress.isPressed() == false) {  
                handleProgress.sendEmptyMessage(0);  
            }  
        }  
    };  
      
    Handler handleProgress = new Handler() {  
        public void handleMessage(Message msg) {  
  
            int position = mediaPlayer.getCurrentPosition();  
            int duration = mediaPlayer.getDuration();  
              
            if (duration > 0) {  
                long pos = skbProgress.getMax() * position / duration;  
                skbProgress.setProgress((int) pos);  
            }  
        };  
    };  
    //*****************************************************  
      
    public void play()  
    {  
        mediaPlayer.start();  
    }  
      
    public void playUrl(String videoUrl)  
    {  
        try {  
            mediaPlayer.reset();  
            mediaPlayer.setDataSource(videoUrl);  
            mediaPlayer.prepare();//prepare之后自动播放  
            //mediaPlayer.start();  
        } catch (IllegalArgumentException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        } catch (IllegalStateException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        } catch (IOException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
    }  
  
      
    public void pause()  
    {  
        mediaPlayer.pause();  
    }  
      
    public void stop()  
    {  
        if (mediaPlayer != null) {   
            mediaPlayer.stop();  
            mediaPlayer.release();   
            mediaPlayer = null;   
        }   
    }  
  
    @Override  
    /**  
     * 通过onPrepared播放  
     */  
    public void onPrepared(MediaPlayer arg0) {  
        arg0.start();  
        Log.e("mediaPlayer", "onPrepared");  
    }  
  
    @Override  
    public void onCompletion(MediaPlayer arg0) {  
        Log.e("mediaPlayer", "onCompletion");  
    }  
  
    @Override  
    public void onBufferingUpdate(MediaPlayer arg0, int bufferingProgress) {  
        skbProgress.setSecondaryProgress(bufferingProgress);  
        int currentProgress=skbProgress.getMax()*mediaPlayer.getCurrentPosition()/mediaPlayer.getDuration();  
        Log.e(currentProgress+"% play", bufferingProgress + "% buffer");  
    }  
  
}  
```
`test_musicplayer.java`是主程序，负责调用Player类，其中关键部分是`SeekBarChangeEvent`这个SeekBar拖动的事件：SeekBar的Progress是0~SeekBar.getMax()之内的数，而MediaPlayer.seekTo()的参数是0~MediaPlayer.getDuration()之内数，所以MediaPlayer.seekTo()的参数是`(progress/seekBar.getMax())*player.mediaPlayer.getDuration()`。

`test_musicplayer.java`源码如下：

```java
package com.musicplayer;  
  
import android.app.Activity;  
import android.os.Bundle;  
import android.view.View;  
import android.view.View.OnClickListener;  
import android.widget.Button;  
import android.widget.SeekBar;  
  
public class test_musicplayer extends Activity {  
    private Button btnPause, btnPlayUrl, btnStop;  
    private SeekBar skbProgress;  
    private Player player;  
  
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
      
        this.setTitle("在线音乐播放---hellogv编写");  
  
        btnPlayUrl = (Button) this.findViewById(R.id.btnPlayUrl);  
        btnPlayUrl.setOnClickListener(new ClickEvent());  
  
        btnPause = (Button) this.findViewById(R.id.btnPause);  
        btnPause.setOnClickListener(new ClickEvent());  
  
        btnStop = (Button) this.findViewById(R.id.btnStop);  
        btnStop.setOnClickListener(new ClickEvent());  
  
        skbProgress = (SeekBar) this.findViewById(R.id.skbProgress);  
        skbProgress.setOnSeekBarChangeListener(new SeekBarChangeEvent());  
        player = new Player(skbProgress);  
  
    }  
  
    class ClickEvent implements OnClickListener {  
  
        @Override  
        public void onClick(View arg0) {  
            if (arg0 == btnPause) {  
                player.pause();  
            } else if (arg0 == btnPlayUrl) {  
                //在百度MP3里随便搜索到的,大家可以试试别的链接  
                String url="http://219.138.125.22/myweb/mp3/CMP3/JH19.MP3";  
                player.playUrl(url);  
            } else if (arg0 == btnStop) {  
                player.stop();  
            }  
        }  
    }  
  
    class SeekBarChangeEvent implements SeekBar.OnSeekBarChangeListener {  
        int progress;  
  
        @Override  
        public void onProgressChanged(SeekBar seekBar, int progress,  
                boolean fromUser) {  
            // 原本是(progress/seekBar.getMax())*player.mediaPlayer.getDuration()  
            this.progress = progress * player.mediaPlayer.getDuration()  
                    / seekBar.getMax();  
        }  
  
        @Override  
        public void onStartTrackingTouch(SeekBar seekBar) {  
  
        }  
  
        @Override  
        public void onStopTrackingTouch(SeekBar seekBar) {  
            // seekTo()的参数是相对与影片时间的数字，而不是与seekBar.getMax()相对的数字  
            player.mediaPlayer.seekTo(progress);  
        }  
    }  
  
}  
```


> CSDN