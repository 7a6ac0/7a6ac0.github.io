---
layout:     post
title:      "[Kotlin] 使用Exoplayer播放器播放影片"
subtitle:   "Use Exoplayer to play video with kotlin."
date:       2018-01-11 15:10:00
author:     "Tabaco"
header-img: "img/in-post/post-exoplayer-kotlin/post-bg-exoplayer-kotlin.png"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - Kotlin
---

## 什麼是Exoplayer?
Android內提供的mediaPlayer只支援有限的格式，例如影片雖支持MP4，3GP，但如果要播放其他格式的影片還要進行相應的轉換。Exoplayer支援基於HTTP的動態自適應流 ([DASH][i1]) 和SmoothStreaming，任何目前MediaPlayer 支持的視頻格式，同時它還支持 HTTP 直播(HLS)，MP4，MP3，WebM，M4A，MPEG-TS 和 AAC。

相關的網站如下：
* Exoplayer官網：[http://google.github.io/ExoPlayer/][i2]
* Github：[https://github.com/google/ExoPlayer][i3]
* 開發指南：[http://google.github.io/ExoPlayer/guide.html][i4]

## Exoplayer基本設定
1. Exoplayer發布在Jcenter上，所以需要在專案根目錄的`build.gralde`包含jcenter repository。
```
allprojects {
    repositories {
        jcenter()
    }
}
```
2. 添加Exoplayer的compile dependency到你app的`build.gradle`文件內。
```
compile 'com.google.android.exoplayer:exoplayer:r2.X.X'
```
3. 或者只添加需要的dependency，例如想要開發只支援DASH影片播放的APP，則只需要添加core、dash及UI即可。
```
compile 'com.google.android.exoplayer:exoplayer-core:r2.X.X'
compile 'com.google.android.exoplayer:exoplayer-dash:r2.X.X'
compile 'com.google.android.exoplayer:exoplayer-ui:r2.X.X'
```

    其中2.X.X改成你專案需要的版本，在這篇是使用2.6.1。
    Exoplayer可用的module包括如下：
    * exoplayer-core：Exoplayer核心功能，所以必須添加。
    * exoplayer-dash：支援DASH格式。
    * exoplayer-hls：支援HLS格式。
    * exoplayer-smoothstreaming：支援SmoothStreaming格式。
    * exoplayer-ui：UI相關資源。

    可到[binary][i5]上查看更多資訊。

## 建立Exoplayer APP

1. 在XML內新增Exoplayer的view(SimpleExoPlayerView)，並為了顯示正在讀取影片，這在多新增一個`ProgressBar`。
    ```xml
    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/video_linearlayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".VideoViewActivity">

        <FrameLayout
            android:id="@+id/video_framelayout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:keepScreenOn="true">

            <com.google.android.exoplayer2.ui.SimpleExoPlayerView
                android:id="@+id/simpleExoPlayerView"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:focusable="true" />

            <ProgressBar
                android:id="@+id/video_progressbar"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:visibility="invisible" />
        </FrameLayout>
    </LinearLayout>
    ```
2. 初始化播放器
```kotlin
private lateinit var videoFrameLayout: FrameLayout
private lateinit var simpleExoPlayerView: SimpleExoPlayerView
private lateinit var videoProgressbar: ProgressBar
private lateinit var simpleExoplayer: SimpleExoPlayer
private val url = "http://demos.webmproject.org/exoplayer/glass.mp4"
private var mResumePosition = 0L
private val bandwidthMeter by lazy {
        DefaultBandwidthMeter()
}
private val adaptiveTrackSelectionFactory by lazy {
        AdaptiveTrackSelection.Factory(bandwidthMeter)
}
private fun initializeExoplayer() {
        simpleExoplayer = ExoPlayerFactory.newSimpleInstance(
                DefaultRenderersFactory(this),
                DefaultTrackSelector(adaptiveTrackSelectionFactory),
                DefaultLoadControl()
        )

        videoFrameLayout = findViewById(R.id.video_framelayout)
        simpleExoPlayerView = findViewById<SimpleExoPlayerView>(R.id.simpleExoPlayerView).apply {
            player = simpleExoplayer
        }
        videoProgressbar = findViewById(R.id.video_progressbar)

        simpleExoplayer.seekToDefaultPosition()
        simpleExoplayer.prepare(buildMediaSource(Uri.parse(url)))
        simpleExoplayer.playWhenReady = true
        simpleExoplayer.addListener(playerEventListener)
    }
```

3. 準備Player
在ExoPlayer中，任何播放媒體均由MediaSource來表示，如要播放一塊媒體，必須先創建一個相對應的MediaSource，然後將此對像傳遞給simpleExoplayer.prepare。
ExoPlayer庫為DASH(DashMediaSource)，SmoothStreaming(SsMediaSource)，HLS(HlsMediaSource)和常規媒體文件(ExtractorMediaSource)都提供了MediaSource的實現。以下顯示如何使用建立適合播放MP4文件的MediaSource。
```kotlin
private fun buildMediaSource(uri: Uri) : MediaSource {
        val dataSourceFactory = DefaultDataSourceFactory(this, Util.getUserAgent(this, "ExoplayerSample"), bandwidthMeter)
        return ExtractorMediaSource.Factory(dataSourceFactory).createMediaSource(uri)
    }
```

4. 監聽播放器的狀態
讓影片尚未播放前顯示正在讀取影片，而在可以播放狀態下隱藏讀取狀態，讓使用者得知可以觀看影片，因此透過Player.EventListener的onPlayerStateChanged更改ProgressBar的狀態。
```kotlin
private val playerEventListener by lazy {
        PlayerEventListener()
}
inner private class PlayerEventListener : Player.EventListener {
        override fun onPlaybackParametersChanged(playbackParameters: PlaybackParameters?) {
        }

        override fun onSeekProcessed() {
        }

        override fun onTracksChanged(trackGroups: TrackGroupArray?, trackSelections: TrackSelectionArray?) {
        }

        override fun onPlayerError(error: ExoPlaybackException?) {
        }

        override fun onLoadingChanged(isLoading: Boolean) {
        }

        override fun onPositionDiscontinuity(reason: Int) {
        }

        override fun onRepeatModeChanged(repeatMode: Int) {
        }

        override fun onShuffleModeEnabledChanged(shuffleModeEnabled: Boolean) {
        }

        override fun onTimelineChanged(timeline: Timeline?, manifest: Any?) {
        }

        override fun onPlayerStateChanged(playWhenReady: Boolean, playbackState: Int) {
            when(playbackState) {
                Player.STATE_BUFFERING -> {
                    videoProgressbar.visibility = View.VISIBLE
                }
                Player.STATE_READY -> {
                    videoProgressbar.visibility = View.INVISIBLE
                }
            }
        }
    }
```

5. 紀錄上次播放時間點
無論是關閉手機螢幕或按HOME鍵，APP均會進入onPause狀態，為了使重新開啟APP時回到上次時間點，必須記錄上次影片時間點。而在APP進入onDestroy時釋放Exoplayer。
```kotlin
    override fun onResume() {
        simpleExoplayer.seekTo(mResumePosition)
        super.onResume()
    }

    override fun onPause() {
        pauseExoplayer()
        super.onPause()
    }

    override fun onDestroy() {
        releaseExoplayer()
        super.onDestroy()
    }
    private fun pauseExoplayer() {
        simpleExoplayer.playWhenReady = false
        mResumePosition = simpleExoplayer.currentPosition
    }

    private fun releaseExoplayer() {
        if (simpleExoplayer != null)
            simpleExoplayer.release()
    }
```

以上設定完就可以正常播放影片了。


[i1]: https://zh.wikipedia.org/wiki/%E5%9F%BA%E4%BA%8EHTTP%E7%9A%84%E5%8A%A8%E6%80%81%E8%87%AA%E9%80%82%E5%BA%94%E6%B5%81
[i2]: http://google.github.io/ExoPlayer/
[i3]: https://github.com/google/ExoPlayer
[i4]: http://google.github.io/ExoPlayer/guide.html
[i5]: https://bintray.com/google/exoplayer