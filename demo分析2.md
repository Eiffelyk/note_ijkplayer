# demo分析(2)
___
##描述
##IjkVideoView最简逻辑分析
上一篇实现了最简demo，除了初始化调用native外，只需要初始化IjkVideoView，setVideoPath()，start() 就可以完成视频的播放
### IjkVideoView初始化
###### 代码调用逻辑
```java 
IjkVideoView{ //构造函数
   initVideoView{ //初始化videoView
      ...initBackground() //初始化后台播放
      initRenders{
            setRender{
               setRenderView();
            }
         }
      }
      ...获取焦点，设置状态，添加字母显示等
   }
}
```
###### 说明
* 初始化构建了一个用于承载视频内容的视图view。
* 并根据是否是后台播放【高版本无法使用】
* 并根据设置使用SurfaceView/TextureView/View进行渲染
* 最后又给IjkVideoView设置了焦点事件，触屏事件，以及添加了字幕子view
### setVideoPath() 分析
###### 代码调用逻辑
```java 
   setVideoPath{
      setVideoURI{
         openVideo{
            MediaPlayer mMediaPlayer = createPlayer() //此处根据setting中的设置初始化不同的player
            mMediaPlayer.setDataSource()//调用native的_setDataSource方法将视频源赋值给native层播放器
            bindSurfaceHolder()//播放器和视图进行绑定
            ...mMediaPlayer的其他设置
            attachMediaController() //控制器和播放器绑定
         }
         requestLayout();
         invalidate();
      }
   }
```
###### 说明
1. setVideoPath,setVideoURI都进行了重载，来不同类型地址，实现播放器可以播放不同方式源，最后通过setVideoURI将转成Uri
2. 并根据设置使用SurfaceView/TextureView/View进行渲染
3. 最后又给IjkVideoView设置了焦点事件，触屏事件，以及添加了显示字幕子view
### openVideo() 关键点
   * 初始化音频系统，
   * 构造了player
   * player设置各种事件监听【将native的状态信息传递给上层】
   * 将视频源设置给播放器
   ```java
       //支持的视频源类型
       void setDataSource(Context context, Uri uri)
               throws IOException, IllegalArgumentException, SecurityException, IllegalStateException;
   
       @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
       void setDataSource(Context context, Uri uri, Map<String, String> headers)
               throws IOException, IllegalArgumentException, SecurityException, IllegalStateException;
   
       void setDataSource(FileDescriptor fd)
               throws IOException, IllegalArgumentException, IllegalStateException;
   
       void setDataSource(String path)
               throws IOException, IllegalArgumentException, SecurityException, IllegalStateException;
       /*--------------------
        * AndroidMediaPlayer: M:
        */
       void setDataSource(IMediaDataSource mediaDataSource);
   ```
   * 将view和播放器进行绑定
   * 将控制和播放器进行绑定
### createPlayer(int playerType)
   ```java 
   public IMediaPlayer createPlayer(int playerType) {
        IMediaPlayer mediaPlayer = null;

        switch (playerType) {
            case Settings.PV_PLAYER__IjkExoMediaPlayer: {
                IjkExoMediaPlayer IjkExoMediaPlayer = new IjkExoMediaPlayer(mAppContext);
                mediaPlayer = IjkExoMediaPlayer;
            }
            break;
            case Settings.PV_PLAYER__AndroidMediaPlayer: {
                AndroidMediaPlayer androidMediaPlayer = new AndroidMediaPlayer();
                mediaPlayer = androidMediaPlayer;
            }
            break;
            case Settings.PV_PLAYER__IjkMediaPlayer:
            default: {
                IjkMediaPlayer ijkMediaPlayer = null;
                if (mUri != null) {
                    ijkMediaPlayer = new IjkMediaPlayer();
                    ...各种设置
                    }
                mediaPlayer = ijkMediaPlayer;
            }
            break;
        }

        if (mSettings.getEnableDetachedSurfaceTextureView()) {
            mediaPlayer = new TextureMediaPlayer(mediaPlayer);
        }

        return mediaPlayer;
    }
   ```
   * switch(playerType)根据setting中不同类型确定的，支持ijkPalyer，android原生，ExoMediaPlayer三种
   * 如果是ijkmediaPlayer进行各种参数设置，这些参数这会映射到native层，后续单开一篇将各个参数的设置
   * 是否使用实现了IMediaPlayer, ISurfaceTextureHolder的播放器
### start()  (pause逻辑相同)
###### 代码调用逻辑
```java
if (isInPlaybackState()) {
            mMediaPlayer.start(); //调用native的_start方法进行播放
            mCurrentState = STATE_PLAYING;
        }
        mTargetState = STATE_PLAYING;
}
```
###### 说明
这个部分没有什么可说的，简单来说就是调用native层对应的方法，然后设置播放状态
### 结束播放 
###### 代码调用逻辑
```java
stopPlayback() //针对前台播放的情况
release()
//或者
stopBackgroundPlay() //针对在后台播放的情况
```
###### 说明
* 停止的时候需要调用release进行资源释放
* stopPlayback和release对比发现重复度非常高。可以根据需求自己修改

####到此你会你简单使用ijkplayer
