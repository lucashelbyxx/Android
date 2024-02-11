[Get started with the NDK  | Android NDK  | Android Developers](https://developer.android.com/ndk/guides)



# NDK (Native Development Kit)

## 1. 入门

### NDK 简介

NDK (Native Development Kit)，使能够在 Android 应用中嵌入C 和 C++ 代码。提供了 [platform libraries](https://developer.android.com/ndk/guides/stable_apis) 可使用这些平台库管理原生 activity 和访问实体设备组件，例如传感器和触控输入。

如果需要实现以下一个或多个目标，NDK 就能派上用场：

- 平台之间移植其应用

- 进一步提升设备性能，以降低延迟或运行游戏或物理模拟等计算密集型应用。
- 复用 C 或 C++ 库。

使用 NDK 将 C 和 C++ 代码编译到原生库中，然后使用 Android Studio 的集成构建系统 Gradle 将原生库打包到 APK 中。Java 代码随后可以通过 [Java 原生接口 (JNI)](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html) 框架调用原生库中的函数。



### 运作方式

Android 构建原生应用时使用的主要组件，以及构建和封装过程。

**主要组件**

- 原生共享库：NDK 从 C/C++ 源代码构建这些库或 .so 文件
- 原生静态库：NDK 也可构建静态库或 .a 文件，可将静态库关联到其他库
- Java 原生接口 (JNI)：JNI 是 Java 和 C++ 组件用于相互通信的接口。[Java 原生接口规范](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)。
- 应用二进制接口 (ABI)：ABI 可以非常精确地定义应用的机器代码在运行时应该如何与系统交互。NDK 根据这些定义构建 `.so` 文件。不同的 ABI 对应不同的架构：NDK 为 32 位 ARM、AArch64、x86 及 x86-64 提供 ABI 支持。如需了解详情，请参阅 [Android ABI](https://developer.android.com/ndk/guides/abis?hl=zh-cn)。
- 清单：如果您编写的应用不包含 Java 组件，必须在[清单](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=zh-cn)中声明 [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=zh-cn) 类。要详细了解如何执行此操作，请参阅[使用 native_activity.h 接口](https://developer.android.com/ndk/guides/concepts?hl=zh-cn#na)。



**Android 开发原生应用一般流程** 

1. 设计应用，确定 Java 实现部分，以及原生代码实现的部分
2. 创建 Android 应用项目
3. 若要编写纯原生应用，要在 `AndroidManifest.xml` 中声明 [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=zh-cn) 类。[原生 activity 和应用](https://developer.android.com/ndk/guides/concepts?hl=zh-cn#naa)。
4. 在“JNI”目录中创建一个描述原生库（包括名称、标记、关联库和要编译的源文件）的 `Android.mk` 文件。
5. 或者，您也可以创建一个配置目标 ABI、工具链、发布/调试模式和 STL 的 `Application.mk` 文件。对于您未指定的任何属性，系统将分别使用以下默认值：
   - ABI：所有非废弃的 ABI
   - 模式：发布
   - STL：系统
6. 将原生源代码放在项目的 `jni` 目录下。
7. 使用 ndk-build 编译原生（`.so`、`.a`）库。
8. 构建 Java 组件，生成可执行 `.dex` 文件。
9. 将所有内容封装到一个 APK 文件中，包括 `.so`、`.dex` 以及应用运行所需的其他文件。







### NDK 使用

[NDK 使用入门](https://developer.android.com/ndk/guides?hl=zh-cn)



### 中间件建议

[针对中间件供应商的建议  | Android NDK  | Android Developers](https://developer.android.com/ndk/guides/middleware-vendors?hl=zh-cn)



## 2. NDK 构建



## 3. C/C++ 代码

### **注意事项**

- 静态运行时

  如果应用的所有原生代码均位于一个共享库中，建议使用静态运行时。这样可让链接器最大限度内联和精简未使用的代码，使应用达到最优化状态且文件最小巧。这样做还能避免旧版 Android 中的 PackageManager 和动态链接器出现错误，此类错误可导致处理多个共享库变得困难，且容易出错。

  然而，在 C++ 中，在单一程序中定义多个相同函数或对象的副本并不安全。这是 C++ 标准中提出的[单一定义规则](http://en.cppreference.com/w/cpp/language/definition)的一个方面。

  如果使用静态运行时（以及一般静态库），很容易在不经意间破坏这条规则。例如，以下应用就破坏了这一规则：

  ```makefile
  # Application.mk
  APP_STL := c++_static
  # Android.mk
  
  include $(CLEAR_VARS)
  LOCAL_MODULE := foo
  LOCAL_SRC_FILES := foo.cpp
  include $(BUILD_SHARED_LIBRARY)
  
  include $(CLEAR_VARS)
  LOCAL_MODULE := bar
  LOCAL_SRC_FILES := bar.cpp
  LOCAL_SHARED_LIBRARIES := foo
  include $(BUILD_SHARED_LIBRARY)
  ```

  在这种情况下，包括全局数据和静态构造函数在内的 STL 将同时存在于两个库中。此应用的运行时行为未定义，因此在实际运行过程中，应用会经常崩溃。其他可能存在的问题包括：

  - 内存在一个库中分配，而在另一个库中释放，从而导致内存泄漏或堆损坏。
  - `libfoo.so` 中引发的异常在 `libbar.so` 中未被捕获，从而导致应用崩溃。
  - `std::cout` 的缓冲未正常运行。

  将静态运行时链接至多个库，除了会导致行为问题，还会在每个共享库中复制代码，从而增加应用的大小。

  一般情况下，只有在应用中有且只有一个共享库时，才能使用 C++ 运行时的静态变体。

- 动态运行时

  如果应用包括多个共享库，应使用 `libc++_shared.so`。旧版 Android 的 PackageManager 和动态链接器存在 bug，导致原生库的安装、更新和加载不可靠。具体而言，如果应用以早于 Android 4.3（Android API 级别 18）的 Android 版本为目标，并且您使用 `libc++_shared.so`，必须先加载共享库，再加载依赖于共享库的其他库。

  [ReLinker](https://github.com/KeepSafe/ReLinker) 项目能够解决所有已知的原生库加载问题。

  一个应用不得使用多个 C++ 运行时。不同的 STL 互**不**兼容。举例来说，libc++ 中 `std::string` 的布局不同于 gnustl。根据某种 STL 编写的代码无法使用以另一种 STL 编写的对象。以上仅举一例，其他不兼容情况不胜枚举。

- 每个应用一个 STL



### 原生 API

#### 使用原生 API

#### Core C/C++

- C library
- C++ library
- Logging
- Trace
- zlib



#### Graphics

- OpenGL ES

- EGL
- Vulkan
- Bitmaps
- Sync API

- Camera

- Media

- native application APIs

  如需了解详情，请参阅 [Android NDK API 参考文档](https://developer.android.com/ndk/reference?hl=zh-cn)。

  API 包括：

  - [资源](https://developer.android.com/ndk/reference/group/asset?hl=zh-cn)
  - [Choreographer](https://developer.android.com/ndk/reference/group/choreographer?hl=zh-cn)
  - [配置](https://developer.android.com/ndk/reference/group/configuration?hl=zh-cn)
  - [输入](https://developer.android.com/ndk/reference/group/input?hl=zh-cn)
  - [Looper](https://developer.android.com/ndk/reference/group/looper?hl=zh-cn)
  - [原生 Activity](https://developer.android.com/ndk/reference/group/native-activity?hl=zh-cn)
  - [原生硬件缓冲区](https://developer.android.com/ndk/reference/group/a-hardware-buffer?hl=zh-cn)
  - [原生窗口](https://developer.android.com/ndk/reference/group/a-native-window?hl=zh-cn)
  - [内存](https://developer.android.com/ndk/reference/group/memory?hl=zh-cn)
  - [网络](https://developer.android.com/ndk/reference/group/networking?hl=zh-cn)
  - [传感器](https://developer.android.com/ndk/reference/group/sensor?hl=zh-cn)
  - [存储](https://developer.android.com/ndk/reference/group/storage?hl=zh-cn)
  - [SurfaceTexture](https://developer.android.com/ndk/reference/group/surface-texture?hl=zh-cn)

  Hardware Buffer APIs

- Audio

  - AAudio

    AAudio 是当前支持的原生音频 API。它替换了 OpenSL ES，能更好地支持需要低延迟音频的高性能音频应用。

  - OpenSL ES

    OpenSL ES 是另一个原生音频 API





## 4. 调试和性能分析

### 简介

#### 调试原生代码崩溃

原生代码崩溃转储或 Tombstone，[调试 Android 平台原生代码](https://source.android.com/devices/tech/debug/index.html?hl=zh-cn)

常见崩溃类型及其调查方法，请参阅[诊断原生代码崩溃问题](https://source.android.com/devices/tech/debug/native-crash?hl=zh-cn)

[ndk-stack](https://developer.android.com/ndk/guides/ndk-stack?hl=zh-cn) 工具可帮助您对崩溃问题进行符号化处理。您可以按照常规[调试应用](https://developer.android.com/studio/debug?hl=zh-cn)文档中的说明，在 Android Studio 中调试崩溃问题。命令行，可以使用 [ndk-gdb](https://developer.android.com/ndk/guides/ndk-gdb?hl=zh-cn) 从 shell 附加 `gdb` 或 `lldb`。



**应用直接访问 Tombstone（墓碑）轨迹**

从 Android 12（API 级别 31）开始，可以通过 [`ApplicationExitInfo.getTraceInputStream()`](https://developer.android.com/reference/android/app/ApplicationExitInfo?hl=zh-cn#getTraceInputStream()) 方法以[协议缓冲区](https://developers.google.com/protocol-buffers/?hl=zh-cn)的形式访问应用的原生代码崩溃 Tombstone。协议缓冲区使用[此架构](https://android.googlesource.com/platform/system/core/+/refs/heads/main/debuggerd/proto/tombstone.proto)进行序列化。

```java
ActivityManager activityManager: ActivityManager = getSystemService(Context.ACTIVITY_SERVICE);
MutableList<ApplicationExitInfo> exitReasons = activityManager.getHistoricalProcessExitReasons(/* packageName = */ null, /* pid = */ 0, /* maxNum = */ 5);
for (ApplicationExitInfo aei: exitReasons) {
    if (aei.getReason() == REASON_CRASH_NATIVE) {
        // Get the tombstone input stream.
        InputStream trace = aei.getTraceInputStream();
        // The tombstone parser built with protoc uses the tombstone schema, then parses the trace.
        Tombstone tombstone = Tombstone.parseFrom(trace);
    }
}
```



#### *调试原生内存问题

**Address Sanitizer (HWASan/ASan)**

[HWAddress Sanitizer](https://developer.android.com/ndk/guides/hwasan?hl=zh-cn) (HWASan) 和 [Address Sanitizer](https://developer.android.com/ndk/guides/asan?hl=zh-cn) (ASan) 与 Valgrind 类似，但在 Android 上明显速度更快且支持效果更好。

以下是在 Android 上调试内存错误的最佳方法。

**Malloc 调试**

有关 C 库内置的原生内存问题调试选项的完整说明，请参阅 [Malloc 调试](https://android.googlesource.com/platform/bionic/+/main/libc/malloc_debug/README.md)和[使用 libc 回调跟踪原生内存](https://android.googlesource.com/platform/bionic/+/main/libc/malloc_debug/README_api.md)。

**Malloc 钩子**

如果您想构建自己的工具，Android 的 libc 也支持拦截在程序执行期间发生的所有分配/释放调用。有关使用说明，请参阅 [malloc_hooks 文档](https://android.googlesource.com/platform/bionic/+/main/libc/malloc_hooks/README.md)。

**Malloc 统计信息**

Android 支持 `<malloc.h>` 的 [mallinfo(3)](http://man7.org/linux/man-pages/man3/mallinfo.3.html) 和 [malloc_info(3)](http://man7.org/linux/man-pages/man3/malloc_info.3.html) 扩展。

Android 6.0 (Marshmallow) 及更高版本提供 `malloc_info` 功能，Bionic 的 [malloc.h](https://android.googlesource.com/platform/bionic/+/main/libc/include/malloc.h) 头文件中记录了其 XML 架构。

**性能分析**

如需对原生代码进行 CPU 性能分析，请使用 [Simpleperf](https://developer.android.com/ndk/guides/simpleperf?hl=zh-cn)。



### [Android Studio 调试](https://developer.android.com/studio/debug?hl=zh-cn)



### ndk-gdb

NDK 包含一个名为 `ndk-gdb` 的 Shell 脚本，可以启动命令行原生调试会话。



### ndk-stack

借助 `ndk-stack` 工具，可以使用符号来表示来自 [`adb logcat`](https://developer.android.com/tools/help/logcat?hl=zh-cn) 的堆栈轨迹或 `/data/tombstones/` 中的 Tombstone。该工具会将共享库内的任何地址替换为源代码中对应的 `<source-file>:<line-number>`，从而简化调试流程。



### [Native tracing](https://developer.android.com/topic/performance/tracing/custom-events?hl=zh-cn)



### Simpleperf

Android Studio 包含 Simpleperf 的图形前端，[使用 CPU 性能分析器检查 CPU 活动](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn)

使用命令行，可以直接使用 Simpleperf。Simpleperf 是一个通用的命令行 CPU 性能剖析工具，包含在面向 Mac、Linux 和 Windows 的 NDK 中。

**tips**

[Simpleperf 命令和选项参考](https://developer.android.com/ndk/guides/simpleperf-commands?hl=zh-cn)

查找执行时间最长的共享库

查找执行时间最长的函数

查找线程中所用时间的百分比

查找对象模块中所用时间的百分比

了解函数调用的相关性

对使用 Unity 构建的应用进行性能分析



### Wrap shell script

对包含原生代码的应用进行调试和性能剖析时，一些需要在进程启动时便启用的调试工具通常很有用。这要求您以全新的进程来运行应用，而不是从 zygote 克隆。例如：

- 使用 [strace](https://strace.io/) 跟踪系统调用。
- 使用 [malloc debug](https://android.googlesource.com/platform/bionic/+/master/libc/malloc_debug/README.md) 或 [Address Sanitizer (ASan)](https://github.com/google/sanitizers/wiki/AddressSanitizerOnAndroid) 查找内存错误。
- 使用 [Simpleperf](https://developer.android.com/ndk/guides/simpleperf.html?hl=zh-cn) 进行性能剖析。



### GLES layers







## 5. 内存调试

### Address Sanitizer

[Address Sanitizer](https://source.android.com/docs/security/test/asan?hl=zh-cn)

[AddressSanitizer · google/sanitizers Wiki (github.com)](https://github.com/google/sanitizers/wiki/AddressSanitizer)



[Address Sanitizer](https://developer.android.com/ndk/guides/asan?hl=zh-cn)（也称为 ASan）是最早且最广泛使用的工具。它可以在其他工具均不适用的情况下，用于在测试期间发现内存错误，并调试那些只影响旧款设备的问题。**请尽可能使用 HWASan。**

ASan 是一种基于编译器的快速检测工具，用于检测原生代码中的内存错误。ASan 可以检测以下问题：

- 堆栈和堆缓冲区上溢/下溢
- 释放之后的堆使用情况
- 超出范围的堆栈使用情况
- 重复释放/错误释放

ASan 的 CPU 开销约为 2 倍，代码大小开销在 50% 到 2 倍之间，并且内存开销很大。



**示例应用**

[示例应用](https://github.com/android/ndk-samples/tree/main/sanitizers)展示了如何为 asan 配置 [build 变体](https://developer.android.com/studio/build/build-variants?hl=zh-cn)。



### Hardware Address Sanitizer

[HWAddress Sanitizer](https://clang.llvm.org/docs/HardwareAssistedAddressSanitizerDesign.html)

[了解 HWASan 报告](https://source.android.com/docs/security/memory-safety/hwasan-reports?hl=zh-cn)



[Hardware Address Sanitizer](https://developer.android.com/ndk/guides/hwasan?hl=zh-cn)（也称为 HWASan），最适合用于在测试期间发现内存错误。此工具在自动化测试中最为有用，尤其是在您运行模糊测试时。不过，根据应用的性能需求，它也可以在使用 dogfood 设置的高端手机上使用。

HWASan 能检测到 ASan 所能检测到的同一系列错误。此外，HWASan 还可以检测：

- 返回之后的堆栈使用情况



## 6. 高性能音频

Audio latency

Sampling audio

AAudio

OpenSL ES

Native MIDI API



## 7. Images

### Image decoder

NDK [`ImageDecoder`](https://developer.android.com/ndk/reference/group/image-decoder?hl=zh-cn) API 提供了一种标准 API，供 Android C/C++ 应用直接解码图像。开发者不再需要使用 Java API（通过 JNI）或第三方图像解码库。此 API 与 [Bitmap](https://developer.android.com/ndk/reference/group/bitmap?hl=zh-cn) 模块中的编码函数结合使用，可实现以下功能：

- 缩减原生应用和库的大小，因为它们不再需要链接自己的解码库。
- 应用和库可自动从解码库的平台安全更新中受益。
- 应用可以直接将图像解码到其提供的内存中。如果需要的话，应用接下来可以对图像数据进行后处理，并将其传递给 OpenGL 或其绘制代码。



`ImageDecoder` API 实现位于：

- `imagedecoder.h`，适用于解码器
- `bitmap.h`，适用于编码器
- `libjnigraphics.so`

支持以下图片格式：

- JPEG
- PNG
- GIF
- WebP
- BMP
- ICO
- WBMP
- HEIF
- 数字底片（通过 DNG SDK）

Java framework 里基于解码图片构建的对象：

- [`Drawable` objects](https://developer.android.com/reference/android/graphics/drawable/Drawable)
- [`NinePatch`](https://developer.android.com/reference/android/graphics/NinePatch): 如果出现在编码图片中，将忽略 NinePatch 数据块。
- [Bitmap density](https://developer.android.com/reference/android/graphics/Bitmap#getDensity()): `AImageDecoder` 不会根据屏幕的密度自动调整大小，但允许通过 [`AImageDecoder_setTargetSize()`](https://developer.android.com/ndk/reference/group/image-decoder?hl=zh-cn#aimagedecoder_settargetsize) 将其解码为不同大小。
- Animations: 仅对动画 GIF 或 WebP 文件的第一帧进行解码。



**解码图像**

从表示编码图像的某种形式的输入开始解码。`AImageDecoder` 接受多种类型的输入：

- [`AAsset`](https://developer.android.com/ndk/reference/group/asset) (shown below)
- File descriptor
- Buffer

从文件中打开图片 `Asset`，对其进行解码，然后正确处理解码器和素材资源。如需查看渲染已解码图片的示例，请参阅 [Teapot 示例](https://github.com/android/ndk-samples/tree/develop/teapots/image-decoder/src/main/cpp/Texture.cpp#30)。

```c
AAssetManager* nativeManager = AAssetManager_fromJava(env, jAssets);
const char* file = // Filename
AAsset* asset = AAssetManager_open(nativeManager, file, AASSET_MODE_STREAMING);
AImageDecoder* decoder;
int result = AImageDecoder_createFromAAsset(asset, &decoder);
if (result != ANDROID_IMAGE_DECODER_SUCCESS) {
  // An error occurred, and the file could not be decoded.
}

const AImageDecoderHeaderInfo* info = AImageDecoder_getHeaderInfo(decoder);
int32_t width = AImageDecoderHeaderInfo_getWidth(info);
int32_t height = AImageDecoderHeaderInfo_getHeight(info);
AndroidBitmapFormat format =
       (AndroidBitmapFormat) AImageDecoderHeaderInfo_getAndroidBitmapFormat(info);
size_t stride = AImageDecoder_getMinimumStride(decoder);  // Image decoder does not
                                                          // use padding by default
size_t size = height * stride;
void* pixels = malloc(size);

result = AImageDecoder_decodeImage(decoder, pixels, stride, size);
if (result != ANDROID_IMAGE_DECODER_SUCCESS) {
  // An error occurred, and the file could not be decoded.
}

// We’re done with the decoder, so now it’s safe to delete it.
AImageDecoder_delete(decoder);

// The decoder is no longer accessing the AAsset, so it is safe to
// close it.
AAsset_close(asset);

// Draw the pixels somewhere

// Free the pixels when done drawing with them
free(pixels);
```





## Vulkan





