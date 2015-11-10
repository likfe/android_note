## MediaPlayer实例-带播放动画

完成本例子需要了解 Android 的回调和 Android 中的 AnimationDrawable 的使用。

* [简易易懂的android回调的实现](/function/Callback/androidde_hui_diao_ji_zhi.md)

* [Android中的AnimationDrawable的使用](/UI/Animation/Android中的AnimationDrawable的使用.md) 

### 一、自定义的MAudioPlayer类
```java
package cc.lait.yq.youqun.utils;

import android.content.Context;
import android.media.MediaPlayer;
import android.net.Uri;
import android.util.Log;

/**
 * 聊天页 音频播放
 */
public class MAudioPlayer implements MediaPlayer.OnCompletionListener {
    private MediaPlayer mPlayer;
    MCompletion mCompletion;//自定义的回调接口

    public void setMCompletion(MCompletion mCompletion) {
        this.mCompletion = mCompletion;
    }


    public void stop() {
        if (mPlayer != null) {
            this.mPlayer.stop();
            this.mPlayer.release();
            this.mPlayer = null;
        }
    }

    public void play(Context c, String url) {
        stop();
        try {
            mPlayer = MediaPlayer.create(c, Uri.parse(url));
            mPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                @Override
                public void onPrepared(MediaPlayer mp) {
                    mp.start();
                }
            });
            mPlayer.setOnCompletionListener(this);
            //事件监听必须在 prepare 之前调用，否则不会响应
            mPlayer.prepareAsync();
        } catch (IllegalStateException e) {
            Log.e("IOException", e.toString());
        }


    }

    public boolean isPlaying() {
        if (mPlayer != null && mPlayer.isPlaying()) {
            return true;
        } else return false;
    }

    @Override
    public void onCompletion(MediaPlayer mp) {
        Log.i("--media", "complete");
        stop();
        //调用自定义的回调
        mCompletion.Complete();
    }

    //自定义的回调接口
    public interface MCompletion {
        public void Complete();
    }

}

```
### 二、定义播放动画 `audio_paly.xml`
存放位置 ：`/res/drawable/`
```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
                android:oneshot="false"
    >
    <item
        android:drawable="@drawable/voice_playing_l1"
        android:duration="100"/>
    <item
        android:drawable="@drawable/voice_playing_l2"
        android:duration="100"/>
    <item
        android:drawable="@drawable/voice_playing_l3"
        android:duration="100"/>
</animation-list>
```

### 三、使用
下面是我在`ListView`的`Adapter`中的代码段：
```java
//仅仅贴出了部分代码，明白意思即可
MAudioPlayer mAudioPlayer = new MAudioPlayer();


vhr2 = new VHR2();
vhr2.avast = (ImageView) cv.findViewById(R.id.chat_avast_l);
vhr2.audio = (ImageButton) cv.findViewById(R.id.chat_bt_l);
                    
final AnimationDrawable anim = (AnimationDrawable) context.getResources().getDrawable(
                        R.drawable.audio_play);
                vhr2.audio.setImageDrawable(anim);
                final VHR2 finalVhr = vhr2;
                vhr2.audio.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        if (mAudioPlayer.isPlaying()) {
                            mAudioPlayer.stop();
                            anim.stop();
                            finalVhr.audio.setImageDrawable(context.getResources().getDrawable(R.drawable.voice_playing_l1));
                        } else {
                            mAudioPlayer.setMCompletion(new MAudioPlayer.MCompletion() {
                                @Override
                                public void Complete() {
                                    Log.i("Complete","---");
                                    anim.stop();
                                    finalVhr.audio.setImageDrawable(context.getResources().getDrawable(R.drawable.voice_playing_l1));
                                }
                            });
                            finalVhr.audio.setImageDrawable(anim);
                            mAudioPlayer.play(context, item.getBody());
                            anim.start();
                        }
                    }
                });

//定义的ViewHolder
class VHR2 {
        ImageView avast;
        ImageButton audio;
    }
```



