## Android 使用`MediaPlayer`开发时抛`IllegalStateException`

在我开发的语音播放程序中，首次播放语音没问题，第二次播放时就抛出IllegalStateException异常，由于项目时间比较赶，大致查了下，基本明白问题的原因了，自己debug也证实了一些个推论，但最佳的解决方法却未能找到，只有一个自己想到的笨办法，和同样遇到这问题的人分享一下。 

首先要明确`IllegalStateException`这个异常是什么意思，它是指“非法的状态”。据我调查所知，android的`mediaplayer API`中用到了JNI，也就是我们的java代码是要调用native的C++方法的（mediaplayer是用c++实现的），而这里之所以出现这个异常，就是因为我们java里面的mediaplayer对象的状态和native的对象状态发生了不一致。这个问题再stackOverFlow上面有人问过，虽然回答的人没有给出具体的解决方案，但是原因说的很清楚了：`http://stackoverflow.com/questions/15730772/android-java-lang-illegalstateexception-mediaplayer-isplaying/15730932`，回答中也给出了mediaplayer的c++源码：`http://androidxref.com/4.2.2_r1/xref/frameworks/base/media/jni/android_media_MediaPlayer.cpp#380`，对于我来说，异常是发生在调用isPlaying()方法时，所以查看源码的isPlaying方法，有这么一句： 
```c++
sp<MediaPlayer> mp = getMediaPlayer(env, thiz);
   if (mp == NULL ) {
       jniThrowException(env, "java/lang/IllegalStateException", NULL);
        return false;
    }

```
 可见确实是`native`的`mediaplayer`对象为空引起的（但是我本地的java对象确实不为空，至今为查明原因），这里再把我的方法贴出来，根据里面的注释就能很清楚我的问题在哪里，以及解决方法： 
 ```java
 private void doPlayVoice(String src, final VoiceViewHolder vh,
			final boolean isLeft, int position) {
		if (mp == null)
		{
			mp = new MediaPlayer();
		}
		// 为解决第二次播放时抛出的IllegalStateException，这里做了try-catch处理
		boolean isPlaying = false;
		try {
			isPlaying = mp.isPlaying();
		}
		catch (IllegalStateException e) {
			mp = null;
			mp = new MediaPlayer();
		}
		
		if (isPlaying)
		{
			mp.stop();
			mp.release();
			mp = null;
			nowPlayingPosition = -1;
			mp = new MediaPlayer();
			if (lastAnim != null && lastAnim.isRunning())
			{
				animStop(lastAnim, lastVH, lastIsLeft);
			}
		}
		try
		{
			mp.setDataSource(src);
			mp.setOnPreparedListener(new OnPreparedListener() {
				@Override
				public void onPrepared(MediaPlayer mp) {
					mp.start();
				}
			});
			// Prepare to async playing
			mp.prepareAsync();
		} catch (IllegalArgumentException e)
		{
			e.printStackTrace();
		} catch (SecurityException e)
		{
			e.printStackTrace();
		} catch (IllegalStateException e)
		{
			e.printStackTrace();
		} catch (IOException e)
		{
			e.printStackTrace();
		}
		nowPlayingPosition = position;

		final AnimationDrawable animationDrawable = animStart(vh, isLeft);
		mp.setOnCompletionListener(new OnCompletionListener() {
			@Override
			public void onCompletion(MediaPlayer mp)
			{
				/* 在目前的代码结构下，mp.release()生效(即设nativeContext=0)了，mp = null却未生效，
				 * 导致在下一次播放语音（即下次调用doPlayVoice）时，mp对象不为null，而nativeContext
				 * 却已经被release掉了，于是在执行mp.isPlaying()时就发生了IllegalStateException，为什么
				 * 会发生这样【mp.release()生效了，mp = null却未生效】的状况，原因暂未查明，为解决该异常
				 * 在doPlayVoice方法的开头mp.isPlaying()处加上了try-catch语句，发生异常时即执行mp = null;
				 * mp = new MediaPlayer()两句，以恢复mp的状态为正常，效果是一样的。
				 */
				mp.release();
				mp = null;
				animStop(animationDrawable, vh, isLeft);
				nowPlayingPosition = -1;
			}
		});
		lastAnim = animationDrawable;
		lastVH = vh;
		lastIsLeft = isLeft;
	}

 ```
 
 其实就像另外一个stackoverflow中有人说的： 
>“MediaPlayer can be strange though; it's worth playing around with different statements even if the logic already makes sense; I could help you more in this regard if you posted code. 

>For now, you could just use a try-catch statement and put something in the catch to ensure that MediaPlayer is working properly.”

>`（http://stackoverflow.com/questions/12208696/media-player-isplaying-throws-illegal-state-android）`。

>Mediaplayer的状态有时候是很奇怪的，即便我们的代码逻辑已经看着很完善了，还是应该做一些对于各种异常的捕获。 

还有一篇文章挺好，可以更清楚的了解一下android的mediaplayer的状态： 
`http://www.360doc.com/content/12/0703/16/7724936_222044896.shtml`

 **PS**：有些人说是因为多个线程同时调用mediaplayer的关系 ，但我是在UI线程里做的，所以不涉及他们的说法，最终我的解决方法可能未必是最优的，如果有人有更好的方法，也请不吝赐教。
 
 --- 
 2014/08/29
 
 这两天在完善APP时要增加一个功能，于是又把MediaPlayer这块琢磨了一遍，突然找到了解决方法，原因还是之前说的，Native的mp对象和我本地的java对象状态不一致，之前也说了是下面这段逻辑出的问题：
 ```java
 public void onCompletion(MediaPlayer mp)
			{
				/* 在目前的代码结构下，mp.release()生效(即设nativeContext=0)了，mp = null却未生效，
				 * 导致在下一次播放语音（即下次调用doPlayVoice）时，mp对象不为null，而nativeContext
				 * 却已经被release掉了，于是在执行mp.isPlaying()时就发生了IllegalStateException，为什么
				 * 会发生这样【mp.release()生效了，mp = null却未生效】的状况，原因暂未查明，为解决该异常
				 * 在doPlayVoice方法的开头mp.isPlaying()处加上了try-catch语句，发生异常时即执行mp = null;
				 * mp = new MediaPlayer()两句，以恢复mp的状态为正常，效果是一样的。
				 */
				mp.release();
				mp = null;
				animStop(animationDrawable, vh, isLeft);
				nowPlayingPosition = -1;
			}

 ```
 关键就是“mp.release()生效了，但是mp = null却未生效”，其实说法不对，应该说他们都生效了，只不过我之前以为这两句的效果是作用在我本地java的mp对象上的，但是现在想想onCompletion(MediaPlayer mp)这里参数中传来的mp对象应该是Native对象，所以那两句的效果是作用在了native对象上，这也就能说明为什么我本地java对象和native对象不一致了，既然不一致，那我们让它们一致就行，这里我肯定是要release并且置空的，所以把这两句操作的mp对象改一下，当然在开头做的捕获异常的那种方法就可以去掉了，代码完全恢复正常： 
 ```java
 private void doPlayVoice(String src, final VoiceViewHolder vh,
			final boolean isLeft, int position) {
		if (mp == null)
		{
			mp = new MediaPlayer();
		}
                // 这里就直接用mp.isPlaying()，因为不可能再报IllegalArgumentException异常了
		if (mp.isPlaying())
		{
			mp.stop();
			mp.release();
			mp = null;
			nowPlayingPosition = -1;
			mp = new MediaPlayer();
			if (lastAnim != null && lastAnim.isRunning())
			{
				animStop(lastAnim, lastVH, lastIsLeft);
			}
		}
		try
		{
			mp.setDataSource(src);
			mp.setOnPreparedListener(new OnPreparedListener() {
				@Override
				public void onPrepared(MediaPlayer mp) {
					mp.start();
				}
			});
			// Prepare to async playing
			mp.prepareAsync();
		} catch (IllegalArgumentException e)
		{
			e.printStackTrace();
		} catch (SecurityException e)
		{
			e.printStackTrace();
		} catch (IllegalStateException e)
		{
			e.printStackTrace();
		} catch (IOException e)
		{
			e.printStackTrace();
		}
		nowPlayingPosition = position;

		final AnimationDrawable animationDrawable = animStart(vh, isLeft);
		mp.setOnCompletionListener(new OnCompletionListener() {
			@Override
			public void onCompletion(MediaPlayer mp)
			{
				/* 因为我本地java的mp对象是定义的全局变量，所以通过类名.this.mp的方式得到我的对象，而非操作onCompletion(MediaPlayer mp)参数传给我的native对象，这样一来，本地java对象就被销毁了，native对象自然也被销毁了
				 */
				YOUR_CLASS_NAME.this.mp.release();
				YOUR_CLASS_NAME.this.mp = null;
				animStop(animationDrawable, vh, isLeft);
				nowPlayingPosition = -1;
			}
		});
		lastAnim = animationDrawable;
		lastVH = vh;
		lastIsLeft = isLeft;
	}

 ```
 
 > http://lovelease.iteye.com/blog/2105616