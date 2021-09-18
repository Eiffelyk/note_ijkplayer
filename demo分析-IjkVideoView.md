# demo分析-IjkVideoView
___
## 描述
## 显示
之前已经分析过最简单使用方式了，这一篇是是内部实现
做为一个View，肯定是要显示内容的，IjkVideoView显示了基本视频播放窗口的功能。同时此处也只是实现了基础内容，后续的想象空间非常多
* InfoHudView 信息显示
  * 更新信息的方式采用的时候handler间隔500毫秒发送一次。
  * 更新内容的来源为ijkplayer中，使用的是InfoHudViewHolder.setMediaPlayer(IMediaPlayer mp)将player信息传入
    * 由于支持ijkPalyer，android原生，ExoMediaPlayer三种，拿到消息的时候需要进行判断，如果不是ijkPlayer通过MediaPlayerProxy获取到IMediaPlayer
  * 其中加载准备耗时和拖动进度后加载耗时采用独立的updateLoadCost和updateSeekCost这两个函数进行进行infoHudView内的变量更新
* RenderView  视频画面显示
* MediaController 视频控制
  * 外部调用setMediaController(IMediaController controller)进行绑定player和Controller
* TextView subtitleDisplay 字幕显示
  * 字幕内容更新是在OnTimedTextListener接口callback函数中

## 播放功能相关
```java 
        void    start();
        void    pause();
        int     getDuration();
        int     getCurrentPosition();
        void    seekTo(int pos);
        boolean isPlaying();
        int     getBufferPercentage();
        boolean canPause();
        boolean canSeekBackward();
        boolean canSeekForward();

        /**
         * Get the audio session id for the player used by this VideoView. This can be used to
         * apply audio effects to the audio track of a video.
         * @return The audio session, or 0 if there was an error.
         */
        int     getAudioSessionId();
```
## 事件
```java
    //重写View的事件
  @Override
    public boolean onTouchEvent(MotionEvent ev) {
       
    }

    @Override
    public boolean onTrackballEvent(MotionEvent ev) {
       
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
       
    }
    //各种事件监听器
    CompletionListener mCompletionListener
    InfoListener mInfoListener
    ErrorListener mErrorListener
    BufferingUpdateListener mBufferingUpdateListener
    SeekCompleteListener mSeekCompleteListener
    OnTimedTextListener mOnTimedTextListener
```


