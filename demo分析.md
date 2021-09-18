# demo分析(1)
___
##描述
##最简实现视频播放
在ijkplayer-example中新建了一个Activity，实现最简单调用
1. 初始化lib
    1. 初始化
    ```java
   @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        IjkMediaPlayer.loadLibrariesOnce(null);
        IjkMediaPlayer.native_profileBegin("libijkplayer.so");
        setContentView(R.layout.activity_first);
        mVideoView = findViewById(R.id.first_video_view);
    }
   ```
    2. 资源回收
    ```java
   @Override
   protected void onDestroy() {
   super.onDestroy();
   IjkMediaPlayer.native_profileEnd();
   }
   ```
2. 使用widget/media下的IjkVideoView  
   先设置url，然后调用start()方法就可以直接播放了。
      ```java 
   mVideoView.setVideoPath(mVideoPath);
   mVideoView.start();
   ```
3. IjkVideoView中，我们最简实现中未设置HudViewHolder，需要添加 *mHudViewHolder != null* 的判断。否则会直接崩溃
   ```java 
    IMediaPlayer.OnPreparedListener mPreparedListener = new IMediaPlayer.OnPreparedListener() {
        public void onPrepared(IMediaPlayer mp) {
            ...
            if (mHudViewHolder != null) {
                mHudViewHolder.updateLoadCost(mPrepareEndTime - mPrepareStartTime);
            }
            ...
        }
    }
   ```
4. 完整代码
* activity_first.xml
```java 
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".activities.FirstActivity"
    android:orientation="vertical">

    <tv.danmaku.ijk.media.example.widget.media.IjkVideoView
        android:id="@+id/first_video_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="center" />
    <Button
        android:onClick="button"
        android:text="开始"
        android:layout_gravity="bottom|center_horizontal"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</FrameLayout>
```
* FirstActivity.java
```
package tv.danmaku.ijk.media.example.activities;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import tv.danmaku.ijk.media.example.R;
import tv.danmaku.ijk.media.example.widget.media.IjkVideoView;
import tv.danmaku.ijk.media.player.IjkMediaPlayer;

public class FirstActivity extends AppCompatActivity {
    private String mVideoPath = "http://las-tech.org.cn/kwai/las-test_ld500d.flv";
    private IjkVideoView mVideoView;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        IjkMediaPlayer.loadLibrariesOnce(null);
        IjkMediaPlayer.native_profileBegin("libijkplayer.so");
        setContentView(R.layout.activity_first);
        mVideoView = findViewById(R.id.first_video_view);
    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        IjkMediaPlayer.native_profileEnd();
    }
    public void button(View view) {
        if (mVideoView.isPlaying()&&mVideoView.isBackgroundPlayEnabled()) {
            mVideoView.stopBackgroundPlay();
            ((Button)view).setText("开始");
        }else if (mVideoView.isPlaying()&&!mVideoView.isBackgroundPlayEnabled()){
            mVideoView.stopPlayback();
            mVideoView.release(true);
            mVideoView.stopBackgroundPlay();
            ((Button)view).setText("开始");
        }else{
            mVideoView.setVideoPath(mVideoPath);
            mVideoView.start();
            ((Button)view).setText("结束");
        }
    }
}
   
```