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
        mCompletion.Complete();
    }

    //自定义的回调接口
    public interface MCompletion {
        public void Complete();
    }

}

```