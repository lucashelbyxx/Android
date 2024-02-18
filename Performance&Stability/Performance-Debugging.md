[应用性能指南](https://developer.android.com/topic/performance/overview?hl=zh-cn)

[native code 帧速率一致性](https://developer.android.com/games/optimize?hl=zh-cn#framerate-consistency)



![debug-config](assets/debug-config.png)



# 一、性能检查

性能检查着重点：

- 应用启动
- 卡顿（Slow rendering、jank）
- 屏幕转换和导航事件
- 长时间运行的工作
- 后台操作（如 I/O、网络）

也可以检查应用工作流的关键用户场景。



检查方法：

**1、人工检查**

[Quickstart: Record traces on Android](https://perfetto.dev/docs/quickstart/android-tracing)

[Overview of system tracing](https://developer.android.com/topic/performance/tracing)

**2、自动化测试**

除了人工检查外，还可以通过自动化测试来收集和汇总性能数据。可帮助用户查看和确定何时可能衰退。 

[Benchmark your app](https://developer.android.com/topic/performance/benchmarking/benchmarking-overview)



**应用启动性能**

**1、使用基准库了解本地性能**

- [Macrobenchmark 库](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview?hl=zh-cn)可帮助测量更大规模的最终用户互动，例如启动、与界面的互动和动画。
- [Microbenchmark 库](https://developer.android.com/topic/performance/benchmarking/microbenchmark-overview?hl=zh-cn)可帮助分析更精细的应用特定情形下的性能。

**2、了解生产环境性能**

- [Android Vitals](https://developer.android.com/topic/performance/vitals?hl=zh-cn) 可以在各种性能指标超出预定阈值时提醒您，从而帮助您提升应用性能。
- [Firebase 性能 SDK](https://firebase.google.com/docs/perf-mon/get-started-android?hl=zh-cn) 会收集有关应用性能的各种指标。例如，可以使用该 SDK 测量从用户打开应用到应用进入响应状态所用的时间，这有助于发现潜在的启动瓶颈。

**3、Android Studio 本地性能分析**

[Record traces](https://developer.android.com/studio/profile/record-traces)

[Simpleperf](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/README.md), native 堆栈采样工具，分析 Android 应用程序和 Android 上运行的 native  进程。它可以分析 Android 上的 Java 和 C++ 代码。

**4、Perfetto trace**

[overview of Perfetto traces](https://perfetto.dev/docs/)

[Run Perfetto using `adb`](https://developer.android.com/studio/command-line/perfetto)

[Recording a trace through the cmdline](https://perfetto.dev/docs/quickstart/android-tracing#recording-a-trace-through-the-cmdline)

[Perfetto web-based trace viewer](https://perfetto.dev/docs/quickstart/android-tracing#recording-a-trace-through-the-perfetto-ui)



## 1、性能分析和跟踪

### 1）System tracing

**分析 Systrace report**

report element：

- 用户交互

- CPU 活动

- system event

  直方图显示特定的系统级事件，例如纹理计数和特定对象的总大小。SurfaceView 的直方图。该计数表示已传递到显示管道并等待在设备屏幕上显示的组合帧缓冲区的数量。由于大多数设备都是双缓冲或三缓冲的，因此该计数几乎总是 0、1 或 2。

  Surface Flinger 进程的其他直方图（包括 VSync 事件和 UI 线程交换工作）

  ![surface-flinger](assets/surface-flinger.png)

- Display frames

  一条彩色线，后面跟着一堆条形图。这些形状代表已创建的特定线程的状态和帧堆栈。堆栈的每个级别代表对 `beginSection()` 的调用，或者您为应用或游戏定义的自定义跟踪事件的开始。

UI 线程或通常运行应用程序或游戏的主线程始终显示为第一个线程。



**线程状态颜色：**

- 绿色：Running，

  线程正在完成与某个进程相关的工作或正在响应中断。

- 蓝色：Runnable

  线程可以运行但目前未进行调度

- 白色：Sleeping

  线程没有可执行的任务，可能是因为线程在遇到互斥锁定时被阻止。

- 橙色：Uninterruptable sleep 不可中断睡眠

  线程在遇到 I/O 操作时被阻止或正在等待磁盘操作完成。

- 紫色：Interruptable sleep

  线程在遇到另一项内核操作（通常是内存管理）时被阻止



**快捷键**

![perfetto-shortcut](assets/perfetto-shortcut.png)



[定义自定义事件](https://developer.android.com/topic/performance/tracing/custom-events?hl=zh-cn)



### 2）检查 GPU 渲染

[检查 GPU 渲染速度和过度绘制 ](https://developer.android.com/topic/performance/rendering/inspect-gpu-rendering?hl=zh-cn)



## 2、基准化分析

基准测试是检查和监控应用性能的一种方式。定期运行基准测试，以分析和调试性能问题，并帮助确保近期的更改不会引起性能下降。

Android 提供了两种基准测试的库和方法：Macrobenchmark、Microbenchmark。



### Macrobenchmark

[Macrobenchmark](https://developer.android.com/studio/profile/macrobenchmark?hl=zh-cn) 库可衡量更大规模的最终用户互动，例如启动、与界面交互和动画。此库可让您直接控制受测试的性能环境。借助它，还可以通过控制编译、启动和停止应用来直接衡量实际的应用启动或滚动。

Macrobenchmark 库可在外部从您通过测试构建的测试应用注入事件并监控结果。因此，您在编写基准测试时，不要直接调用应用代码，而要像用户那样在应用中操作。



### Microbenchmark

借助 [Microbenchmark](https://developer.android.com/studio/profile/benchmark?hl=zh-cn) 库，您可以直接在一个循环中对应用代码进行基准测试。该库旨在衡量 CPU 工作情况，衡量结果将用于评估最佳情况下的性能（例如，JIT 编译已预热，磁盘访问已缓存），使用内部循环或特定热函数时就可能获得这种最佳性能。此外，该库只能衡量您可以直接单独调用的代码。

如果您的应用需要处理复杂的数据结构，或者采用了一些会在应用运行期间进行多次调用的特定计算密集型算法，就可能适合进行基准测试。您还可以衡量界面的各个部分。例如，您可以衡量绑定一个 `RecyclerView` 的花销、inflate 布局时间，以及从性能的角度来看，`View` 类的“布局和衡量”遍历的要求有多高。

不过，您无法衡量进行基准测试的用例对整体用户体验所产生的影响。在某些情况下，您无法通过基准测试判断卡顿或应用启动时间等瓶颈是否得到了改善。因此，首先使用 [Android 性能分析器](https://developer.android.com/studio/profile?hl=zh-cn)找出这类瓶颈至关重要。找到要调查和优化的代码后，您便可以快速且更轻松地反复运行会进行基准测试的循环，并生成噪声较少的结果，让您能够专注于需要改进的方面。

Microbenchmark 库只能报告应用的相关信息，而非整个系统的相关信息。因此，它最适合用来分析应用在特定情况下的性能，而不是分析系统的整体性能问题。



## 3、衡量性能

### 1）关键性能问题

常见问题

1. 滚动卡顿

   “卡顿” 指画面在以下时候出现断断续续的情况：系统无法及时构建和提供帧，以致无法以请求的频率（60hz 或更高）将其绘制到屏幕上。卡顿问题在滚动操作期间最为明显：本应流畅播放的动画流会出现断断续续的情况。由于应用渲染内容所需的时间超过了相应帧在系统上显示的时长，所以画面移动时就会在一帧或多帧中暂停，导致出现卡顿。

2. 启动延迟

   启动延迟时间是指从点按应用图标、通知或其他入口点到屏幕上显示用户的数据所需的时间。

   争取在应用中实现以下启动目标：

   - 冷启动时间少于 500 ms。当正在启动的应用不存在于系统内存中时，就会发生“冷启动”。应用在系统重新启动后或应用进程被用户/系统停止后首次启动时，就会进行冷启动。

     如果应用已在后台运行，则为“热启动”。冷启动需要系统完成大部分工作，因为系统必须从存储空间加载所有内容并初始化应用。因此，请尽量让冷启动耗时不超过 500 ms。

   - 让 [P95 和 P99](https://www.cnblogs.com/hunternet/p/14354983.html)  延迟时间非常接近延迟时间中值。如果应用需要很长时间才能启动，用户体验会很糟糕。应用启动关键路径中的进程间通信 (IPC) 和不必要的 I/O 可能会遇到锁争用，进而造成不一致的情况。

3. 过渡不流畅

   交互期间出现，例如 tab 切换或加载新的 Activity 时。这些过渡必须是平滑的动画，且没有延迟和视觉闪烁

4. 电源效率低

   创建新对象的内存分配可能是系统中大量工作的原因。因为不仅分配本身需要 AndroidRuntime (ART) 的工作，而且稍后释放这些对象（垃圾回收）也需要花销。分配和收集都更快、更高效，特别是对于临时对象。尽管过去最佳实践是尽可能避免分配对象，但建议采取对您的应用程序和架构最有意义的做法。



### 2）确定问题

1. 检查关键用户流程、场景：
   - 常见启动流程，包括启动器和通知
   - 用户滚动浏览其中数据的屏幕。
   - 在屏幕之间切换。
   - 长时间运行的工作流，例如导航或播放音乐。
2. 使用调试工具检查上述工作中发生的情况：
   - [Perfetto](https://perfetto.dev/)：根据准确的计时数据，确切了解整个设备中的运作情况。
   - [内存分析器](https://developer.android.com/studio/profile/memory-profiler?hl=zh-cn)：了解堆上正在发生的内存分配。
   - [Simpleperf](https://developer.android.com/ndk/guides/simpleperf?hl=zh-cn)：显示有关特定时间段内哪些函数调用占用最多 CPU 的火焰图。当您在 Systrace 中发现有些操作用时很长但又不知道什么原因时，Simpleperf 可以提供额外的信息。



### 3）设置应用进行性能分析

正确设置对于从应用获得准确、可重复、可操作的 benchmark 至关重要。在尽可能接近生产的系统上进行测试，同时抑制干扰。

- Tracepoint

  应用可以利用[自定义 trace 事件](https://developer.android.com/topic/performance/tracing/custom-events?hl=zh-cn)在代码中插桩。在捕获轨迹时，每个部分的跟踪会产生少量开销（约 5 us），因此请勿在每个方法中都使用 trace。跟踪较大的工作块（> 0.1 ms）可以提供关于瓶颈的重要洞察。

- APK 注意事项

  不要在 debug 版本测试性能。debug variant 有助于对堆栈样本进行故障排除和符号化，但它们会对性能产生严重的非线性影响。使用 production-grade code shrinking 配置，某些 ProGuard 配置会删除 tracepoints，进行测试时需要考虑删除相关配置规则。

- System 注意事项

- 应用启动缓慢

  trampoline activity：一个 `activityStart` 后面紧跟着另一个 `activityStart` ，第一个活动不会绘制任何帧。

- 不必要的分配触发频繁 GC

  垃圾回收 (GC) 的频率高于预期。示例显示，在长时间运行的操作期间，每 10 秒就出现一次垃圾回收，表示应用可能遭到不必要地分配，但一直在持续进行：

  ![trace_gc](assets/trace_gc.png)

  在使用内存分析器时，绝大部分的分配都源于某个特定的调用堆栈。不需要激进地消除所有分配，因为这样做可能会使代码维护起来更加困难。不妨改为从分配热点着手。

- Janky frames

  graphics pipeline 相对复杂，并且在确定用户最终是否会看到丢帧时可能存在一些细微差别。某些情况下，platform 使用 buffering 来 rescue a frame。但是，你可以忽略大部分细微差别，从应用程序的角度识别有问题的 frames。

  在绘制帧时几乎不需要应用完成什么工作的情况下，`Choreographer.doFrame()` tracepoints 的间隔在帧速率为 60 FPS的设备上是 16.7 ms：

  ![trace_frames1](assets/trace_frames1.png)

  缩小并浏览轨迹，有时会看到一些帧需要稍长时间才能完成，但这种情况仍然没有问题，因为这些帧的用时并未超过分配给它们的 16.7 ms：

  ![trace_frames2](assets/trace_frames2.png)

  如果您在这种规律的间隔中看到一处中断，那就是卡顿帧

  ![trace_frames3](assets/trace_frames3.png)

- 场景 RecycleView 错误

  不必要的情况下，让 [`RecyclerView`](https://developer.android.com/jetpack/androidx/releases/recyclerview?hl=zh-cn) 的整个后备数据失效可能会导致帧的渲染时间较长，造成卡顿。您应当仅让更改的数据失效，从而最大限度地减少需要更新的视图数量。[呈现动态数据](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView?hl=zh-cn#presenting-dynamic-data)，了解避免高开销 `notifyDatasetChanged()` 调用的方式，这些方式会更新内容，而不是完全替换内容。

  如果您无法正确支持每个嵌套的 `RecyclerView`，就会导致内部 `RecyclerView` 每次都要完全重新创建。每个嵌套的内部 `RecyclerView` 都必须设置一个 [`RecycledViewPool`](https://developer.android.com/reference/kotlin/androidx/recyclerview/widget/RecyclerView.RecycledViewPool?hl=zh-cn)，以便确保能够在每个内部 `RecyclerView` 之间回收视图。

  如果未预提取足够的数据，或未及时预提取，用户可能就需要等待从服务器提取更多数据，才能顺畅滚动到列表底部。尽管从技术角度而言这并不属于卡顿，因为并没有错过任何帧的截止时间，但如果您能修改预提取的时间和数量，让用户不必等待数据，用户体验就能得到显著提升。



# 二、性能优化

Android 提供了多种工具和库，可帮助持续改进应用关键时刻的性能。

**基准配置文件**

将基准配置文件实现到您的应用或库中，以最高效的方式提升性能。它可以大幅缩短应用启动时间、加快呈现速度并提高最终用户性能。如需了解详情，请参阅[基准配置文件](https://developer.android.com/topic/performance/baselineprofiles?hl=zh-cn)。

**启动配置文件**

启动配置文件是一项实验性功能，与基准配置文件类似，但应用方式有所不同，并且具有独特的优势。基准配置文件会在应用安装在设备上以后优化性能，而启动配置文件是在编译时应用的。它会提示 R8 缩减器在 DEX 文件中将常用类归为一组。这可以减少应用启动期间的页面故障，从而缩短启动时间。如需了解详情，请参阅 [DEX 布局优化和启动配置文件](https://developer.android.com/topic/performance/baselineprofiles/dex-layout-optimizations?hl=zh-cn)。

**App Startup 库**

借助 [App Startup 库](https://developer.android.com/topic/libraries/app-startup?hl=zh-cn)，库开发者和应用开发者都可以使用 App Startup 库来简化启动序列并优化启动操作。



## 1、Baseline Profiles

性能对业务指标的影响，[Josh](https://developer.android.com/stories/apps/josh?hl=zh-cn)、[Lyft](https://developer.android.com/stories/apps/lyft?hl=zh-cn)、[TikTok](https://developer.android.com/stories/apps/tiktok?hl=zh-cn) 和 [Zomato](https://developer.android.com/stories/apps/zomato?hl=zh-cn) 。



## 2、应用启动

### 1）分析和优化启动步骤

应用程序通常要在启动期间加载对用户必需的资源。非必需资源可以等到启动完成后才加载。

性能权衡考虑的因素：

- 测量每个操作所花费的时间，并识别需要很长时间才能完成的块。
- 确认资源密集型操作对于应用程序启动至关重要。如果操作可以等到应用完整绘制后再执行，则有助于最大限度地减小启动期间的资源限制。
- 确认是否希望相应操作在应用启动期间执行。很多时候，系统可能会从旧版代码或第三方库调用不必要的操作。
- 尽可能将长时间运行的操作移至后台。在启动期间，后台进程仍可能会影响 CPU 使用率。

在对操作进行全面研究后，您可以在所需的加载时间与将其纳入应用启动的必要性之间做出权衡。在更改应用的工作流时，请务必留意可能会发生回归问题或破坏性更改。



### 2）测量和分析主要操作所用时间

查看启动 trace 并测量 `bindApplication` 或 `activityStart` 等主要操作所用的时间。

通过查看应用启动期间所用的总时间，找出具有以下特点的操作：

- 占用大量时间并且可优化的操作。例如，可以查看 [`Choreographer`](https://developer.android.com/reference/android/view/Choreographer?hl=zh-cn) 绘制时间、布局inflate 时间、库加载时间、[`Binder`](https://developer.android.com/reference/android/os/Binder?hl=zh-cn) 事务或资源加载时间。一般，可查看所有耗时超过 20 ms的操作。
- 阻塞主线程。参阅[浏览 Systrace 报告](https://developer.android.com/topic/performance/tracing/navigate-report?hl=zh-cn)。
- 不需要在启动期间运行。
- 可以等到第一帧绘制完毕后再执行。

进一步研究各个跟踪记录，找出性能上有待改进的方面。



**找出主线程上开销大的操作**

避免在主线程上执行开销大的操作（例如文件 I/O 和网络访问）。因为在主线程上执行开销大的操作可能会导致应用无响应，并延迟其他关键操作。[`StrictMode.ThreadPolicy`](https://developer.android.com/reference/android/os/StrictMode.ThreadPolicy?hl=zh-cn) 可以帮助找出在主线程上执行大开销操作的情况。 最佳做法是在调试 build 中启用 [`StrictMode`](https://developer.android.com/reference/android/os/StrictMode?hl=zh-cn)，以尽早发现问题。使用 `StrictMode.ThreadPolicy` 在检测到违反线程规则的行为时触发应用崩溃。



**TTID 和 TTFD**

如需了解应用生成第一帧所用的时间，请测量[初步显示所用时间](https://developer.android.com/topic/performance/vitals/launch-time?hl=zh-cn#time-initial) (TTID)。不过，此指标不一定反映您的应用启动直至用户可开始与应用互动之间的时间。[完全显示所用时间](https://developer.android.com/topic/performance/vitals/launch-time?hl=zh-cn#time-full) (TTFD) 指标在测量和优化拥有完全可用应用状态所需的代码路径方面更为有用。

了解在应用界面完全绘制后的报告策略，参阅[提高启动时间精确度](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-metrics?hl=zh-cn#startup-accuracy)。



### 3）分析整体线程状态

**找出主线程处于休眠状态的主要区块**

如果长时间处于休眠状态，可能是因为应用的主线程要等待作业完成。对于多线程的应用，请确定主线程在等待的线程，并考虑优化相关操作。确保关键路径不会因为不必要的锁争用而出现延迟。



**减少主线程发生阻塞及进入不可中断的休眠状态的情况**

找出主线程每次进入阻塞状态的实例。线程状态时间轴上用橙色指明这种情况。确认这些操作是符合预期的，还是可以避免的，并根据需要进行优化。

与 IO 相关的可中断休眠可能是值得改进的方面。 其他执行 IO 的进程（即使是无关的应用）都能与前台应用在执行的 IO 争用资源。



**缩短启动时间**

在找到可优化之处后，探索采用一些可能的方案来缩短启动时间：

- 通过延迟和异步方式加载内容，缩短 [TTID](https://developer.android.com/topic/performance/vitals/launch-time?hl=zh-cn#time-initial)。
- 尽量减少进行 binder 调用的调用函数。如果这类函数必不可少，请确保优化其调用，可采用缓存值的方式，而不要重复调用，还可以将非阻塞性作业移至后台线程中。
- 为了让应用看起来启动更快，可以尽快对用户显示需要最少渲染的内容，直到界面的其余部分完成加载。
- 创建[启动配置文件](https://developer.android.com/topic/performance/baselineprofiles/overview?hl=zh-cn#startup-profiles)并将其添加到应用。
- 使用 Jetpack [应用启动库](https://developer.android.com/topic/libraries/app-startup?hl=zh-cn)简化应用启动期间的组件初始化。



### 4）分析 UI 性能

应用启动过程包括启动画面和首页的加载时间。优化应用启动，需要 inspect trace 了解绘制界面所用的时间。



**限制初始化工作**

某些帧的加载时间可能比其他帧更长。认为这类帧在应用启动期间的绘制开销很高。

如需优化初始化，请执行以下操作：

- 优先考虑缓慢的布局传递，选择改进这些方面。
- 通过添加[自定义跟踪事件](https://developer.android.com/topic/performance/tracing/custom-events?hl=zh-cn)来分析来自 Perfetto 的 alert，以减少开销较高的绘制和延迟。



**测量帧数据**

可通过多种方式测量帧数据。以下是五种主要的帧数据收集方法：

- 使用 `dumpsys gfxinfo` 进行本地收集：并非 dumpsys 数据中观察到的所有帧都会导致应用呈现速度缓慢，或者对用户造成影响。不过，观察这项指标在不同发布周期之间的变化，有助于了解总体的性能趋势。了解如何使用 `gfxinfo` 和 `framestats` 将界面性能测量值集成到您的测试实践中，请参阅 [Android 应用测试基础知识](https://developer.android.com/training/testing/performance?hl=zh-cn)。
- 使用 [JankStats](https://developer.android.com/topic/performance/jankstats?hl=zh-cn) 进行字段收集：使用 [JankStats 库](https://developer.android.com/studio/profile/jankstats?hl=zh-cn)从应用的特定部分收集帧呈现时间，并记录和分析数据。
- 在测试中使用 Macrobenchmark（在后台使用 Perfetto）
- **[Perfetto FrameTimeline](https://perfetto.dev/docs/data-sources/frametimeline)**：在 Android 12 上，可以从 Perfetto trace 的 [Frame timeline metrics](https://perfetto.dev/docs/data-sources/frametimeline)，了解导致帧丢失的操作。
- 使用 Android Studio Profiler [检测卡顿](https://developer.android.com/studio/profile/jank-detection?hl=zh-cn)



**查看 main activity 的加载时间**

应用的 main activity 可能包含从多个来源加载的大量信息。检查首页 `Activity` 布局，特别是首页 activity 的 [`Choreographer.onDraw`](https://developer.android.com/reference/android/view/Choreographer?hl=zh-cn) 方法。

- 使用 [`reportFullyDrawn`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#reportFullyDrawn()) 向系统报告您的应用现已完全绘制，达到优化目的。
- 使用 [`StartupTimingMetric`](https://developer.android.com/reference/kotlin/androidx/benchmark/macro/StartupTimingMetric?hl=zh-cn) 和 Macrobenchmark 库来测量 activity 和应用启动情况。
- 查看丢帧情况。
- 找出在渲染或测量方面很耗时的布局。
- 找出加载用时较长的资源。
- 找出在启动期间原本没必要 inflate 的布局。

考虑使用以下可优化 main activity 加载时间的解决方案：

- 简化初始布局。参阅[优化布局层次结构](https://developer.android.com/training/improving-layouts/optimizing-layouts?hl=zh-cn)。

- 添加自定义跟踪点，以获取有关丢帧和复杂布局的更多信息。

- 缩减启动期间加载的位图资源的数量和大小。

- 在布局不会立即 `VISIBLE` 的地方使用 [`ViewStub`](https://developer.android.com/reference/android/view/ViewStub?hl=zh-cn)。`ViewStub` 是一个不可见、零大小的 View，可用于在运行时延迟 inflate 布局资源。参阅 [`ViewStub`](https://developer.android.com/reference/android/view/ViewStub?hl=zh-cn)。

  如果使用的是 [Jetpack Compose](https://developer.android.com/jetpack/compose?hl=zh-cn)，则可以使用状态获取与 `ViewStub` 类似的行为，以延迟加载某些组件：



### 5）[app-startup](https://developer.android.com/topic/libraries/app-startup?hl=zh-cn)



## 3、解决常见问题

## 4、最佳实践



# 三、性能监控