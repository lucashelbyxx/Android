[Android 媒体使用入门  | Android media  | Android Developers](https://developer.android.com/media/guides?hl=zh-cn)

[Archive of stories published by google-exoplayer – Medium](https://medium.com/google-exoplayer/archive)

[ExoPlayer](https://exoplayer.dev/)



实现音视频播放功能，使用 Jetpack Media3 库中的 ExoPlayer

实现媒体编辑功能，使用 Jetpack Media3 库中的 Transformer



# Jetpack Media3





# Media3 ExoPlayer 

Jetpack Media3 定义了一个 `Player` 接口，该接口概述了播放视频和音频文件的基本功能。`ExoPlayer` 是 Media3 中此接口的默认实现。



## 一、基础使用

### 1. 创建媒体播放器

Creating a media player

**1）创建 Exoplayer**

```kotlin
val player = ExoPlayer.Builder(context).build()
```

可以在媒体播放器所在的 `Activity`、`Fragment` 或 `Service` 的 `onCreate()` 生命周期方法中创建媒体播放器。

Builder 包含自定义选项，例如：

- [`setAudioAttributes()`](https://developer.android.com/reference/androidx/media3/exoplayer/ExoPlayer.Builder?hl=zh-cn#setAudioAttributes(androidx.media3.common.AudioAttributes,boolean))，用于配置[音频焦点](https://developer.android.com/media/optimize/audio-focus?hl=zh-cn)处理方式
- [`setHandleAudioBecomingNoisy()`](https://developer.android.com/reference/androidx/media3/exoplayer/ExoPlayer.Builder?hl=zh-cn#setHandleAudioBecomingNoisy(boolean))，用于配置音频输出设备断开连接时的播放行为
- [`setTrackSelector()`](https://developer.android.com/reference/androidx/media3/exoplayer/ExoPlayer.Builder?hl=zh-cn#setTrackSelector(androidx.media3.exoplayer.trackselection.TrackSelector))，用于配置[轨道选择](https://developer.android.com/guide/topics/media/exoplayer/track-selection?hl=zh-cn)

Media3 提供了一个 `PlayerView` 界面组件，您可以将其包含在应用的布局文件中。此组件封装了 `PlayerControlView`（用于播放控件）、`SubtitleView`（用于显示字幕）和 `Surface`（用于渲染视频）。



**2）准备播放器**

使用 [`setMediaItem()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#setMediaItem(androidx.media3.common.MediaItem)) 和 [`addMediaItem()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#addMediaItem(androidx.media3.common.MediaItem)) 等方法将[媒体内容](https://developer.android.com/guide/topics/media/exoplayer/media-items?hl=zh-cn)添加到播放列表进行播放。 然后，调用 [`prepare()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#prepare()) 以开始加载媒体并获取必要的资源。

不应在应用在前台运行之前执行这些步骤。如果位于 `Activity` 或 `Fragment` 中，意味着在 API 级别 24 及更高中使用 `onStart()` 或 API 级别 23 及更低的 `onResume()` 中准备播放器。位于 `Service` 中的播放器，可以在 `onCreate()` 中进行准备。



**3）控制播放器**

播放器准备好后，您可以通过对播放器调用以下方法来控制播放：

- [`play()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#play()) 和 [`pause()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#pause())，用于开始和暂停播放
- [`seekTo()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#seekTo(long))，用于跳转到当前媒体项内的某个位置
- [`seekToNextMediaItem()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#seekToNextMediaItem()) 和 [`seekToPreviousMediaItem()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#seekToPreviousMediaItem())，可浏览播放列表

界面组件（如 `PlayerView` 或 `PlayerControlView`）将在绑定到播放器后相应地更新。



**4）释放播放器**

播放需要资源，例如视频解码器，在不再需要播放器时，必须对播放器调用 [`release()`](https://developer.android.com/reference/androidx/media3/common/Player?hl=zh-cn#release()) 来释放资源。

如果播放器位于 `Activity` 或 `Fragment` 中，API  24 及更高的 `onStop()` （API 23 及更低的 `onPause()` 方法）释放播放器。位于 `Service` 的播放器，可以在 `onDestroy()` 中释放播放器。



### 2. 通过 media session 管理播放

Managing playback with a media session

media session 提供了跨进程通信与媒体播放器交互。将 media session 联系讲课播放器允许你在外部进行媒体播放并从外部源接受播放命令。例如移动大屏设备的系统媒体控件集成。

**1）创建 media session**

初始化播放器后创建 MediaSession

```kotlin
val player = ExoPlayer.Builder(context).build()
val mediaSession = MediaSession.Builder(context, player).build()
```

Media3 会自动将 `Player` 的状态与 `MediaSession` 的状态同步。

**2）向其他客户端授权**

客户端应用可以实现[媒体控制器](https://developer.android.com/reference/androidx/media3/session/MediaController?hl=zh-cn)来控制媒体会话的播放。如需接收请求，请在构建 `MediaSession` 时设置[回调](https://developer.android.com/reference/androidx/media3/session/MediaSession.Callback?hl=zh-cn)对象。

当控制器即将连接到您的媒体会话时，会调用 [`onConnect()`](https://developer.android.com/reference/androidx/media3/session/MediaSession.Callback?hl=zh-cn#onConnect(androidx.media3.session.MediaSession,androidx.media3.session.MediaSession.ControllerInfo)) 方法。您可以使用 [`ControllerInfo`](https://developer.android.com/reference/androidx/media3/session/MediaSession.ControllerInfo?hl=zh-cn) 来[接受](https://developer.android.com/reference/androidx/media3/session/MediaSession.ConnectionResult?hl=zh-cn#accept(androidx.media3.session.SessionCommands,androidx.media3.common.Player.Commands))还是[拒绝](https://developer.android.com/reference/androidx/media3/session/MediaSession.ConnectionResult?hl=zh-cn#reject())请求。[Media3 会话演示应用](https://github.com/androidx/media/blob/release/demos/session/src/main/java/androidx/media3/demo/session/PlaybackService.kt#L104)。

连接后，控制器可以向会话发送播放命令。会话会将命令向下委托给 Player。`Player` 接口中定义的播放和播放列表命令由会话自动处理。

例如，其他回调方法允许您处理对[自定义播放命令](https://developer.android.com/guide/topics/media/media3/getting-started/mediasession?hl=zh-cn#adding-custom)和[修改播放列表](https://developer.android.com/reference/androidx/media3/session/MediaSession.Callback?hl=zh-cn#onAddMediaItems(androidx.media3.session.MediaSession,androidx.media3.session.MediaSession.ControllerInfo,java.util.List))的请求。这些回调类似地包含一个 `ControllerInfo` 对象，因此您可以按请求确定访问控制。



### 3. 后台播放媒体内容

Playing media in the background

应用不在前台运行时继续播放媒体内容，`Player` 和 `MediaSession` 应封装在[前台服务](https://developer.android.com/guide/components/foreground-services?hl=zh-cn)中。Media3 提供了 `MediaSessionService` 接口。

1）实现 MeidaSessionService

2）连接到界面

现在，媒体会话位于与播放器界面所在的 `Activity` 或 `Fragment` 不同的 `Service` 中，您可以使用 `MediaController` 将它们关联起来。在界面的 `Activity` 或 `Fragment` 的 `onStart()` 方法中，为 `MediaSession` 创建 `SessionToken`，然后使用 `SessionToken` 构建 `MediaController`。构建 `MediaController` 是异步进行的。



# Media3 Transformer

Transformer 实现高效可靠的媒体编辑。

- 通过剪辑、缩放和旋转修改视频
- 添加叠加层和滤镜等特效
- 处理特殊格式，例如 HDR 和慢动作视频
- 在应用修改后导出媒体项



