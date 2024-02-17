[Android 接口定义语言 (AIDL)  | Background work  | Android Developers](https://developer.android.com/develop/background-work/services/aidl?hl=zh-cn)

[AIDL 概览  1231231123| Android 开源项目  | Android Open Source Project](https://source.android.com/docs/core/architecture/aidl?hl=zh-cn)



[Android IPC: Part 2 – Binder and Service Manager Perspective](https://blog.hacktivesecurity.com/index.php/2020/04/26/android-ipc-part-2-binder-and-service-manager-perspective/)

[Android IPC Communication Mechanisms: Performance, Security, and Complexity - AOSP Insight](https://aospinsight.com/android-ipc-communication-mechanisms-performance-security-and-complexity/)

[App quality  | Android Developers](https://developer.android.com/privacy-and-security/security-tips#IPC)



# IPC 简介

进程间通信是每个操作系统不可或缺的功能。如果进程 A 要和 进程 B 通信（同步、共享数据等），操作系统必须提供执行此操作的功能。

传统 Linux 进程间通信可以通过管道、套接字、共享文件、共享内存等。Android IPC 包括 Intent、Binder、Messenger with a Service 和 BroadcastReceiver。



Linux: https://www.geeksforgeeks.org/inter-process-communication-ipc/
OSX: https://developer.apple.com/documentation/uikit/inter-process_communication
Windows: https://docs.microsoft.com/en-us/windows/win32/ipc/interprocess-communications



## 多进程 Andorid APP

Android 启动的每个应用程序都在自己的进程中运行（具有唯一的进程 ID 或 PID）。这样应用程序会处在一个隔离的环境中，不会受到其他应用、进程的阻碍。通常，启动应用时，Android 会创建一个进程，生成主线程并开始运行 main Activity。

Android 允许指定应用的某些组件，使其在与启动应用的进程不同的进程上运行。可以通过使用 **process** tag 来实现。



**多进程的优势**

1. 内存管理。对于应用程序内存预算，Android 有很大的限制，从最新设备的 24/32/48 MB 到旧设备的 16 MB。RAM 预算必须足以加载类、线程、服务、UI 资源和您的应用程序想要显示的内容。应用程序的每个进程都将具有专用的内存预算。
2. 安全。进程在设计上是隔离的。每个进程都有自己的 Dalvik VM 实例。无法在这些实例之间共享数据。例如，静态字段对于每个进程都有自己的值，而不是所有进程中通用的单个值。



# IPC 机制

Android 为 IPC 提供了各种机制，包括 AIDL、Intent 和 Messenger。每种方法都有自己的优点和缺点。

AIDL、Messenger 和 Intent 各有其性能、安全性和复杂性的权衡。AIDL 提供了一个功能丰富的解决方案，具有强大的性能和安全性，但需要更复杂的实现。Messenger 提供了更简单的实现过程和足够的安全性，但缺乏 AIDL 提供的并发性和一些功能。Intent是最容易实现的，具有合理的安全性，但仅支持单向通信。

**1. AIDL**

AIDL 是一种强大的 IPC 通信机制，提供以下功能：

- 双向通信
- 能够直接从远程进程调用方法，适合共享数据或从另一个进程调用方法
- Priv-app 和系统应用的 SELinux 保护
- Android 应用安全性：通过定义所需权限和使用同一秘钥进行签名的额外保护

AIDL 与其他 IPC 机制相比，实现更加复杂。



**2. Messenger**

Messenger 是另一种 IPC 通信机制，具有以下特点：

- 双向通信
- 非并发操作
- 作为 Binder 的简单封装器构建，底层 IBinder 由 Messenger 使用

Messenger 比 AIDL 更容易实现，支持双向通信。

缺点：非并发操作可能会限制某些场景的使用，功能不如 AIDL 丰富。



**3. Intent**

Intent 是一种易于实现的 IPC 通信机制，提供以下功能：

- 单向通信，支持通过 ResultReceiver 从目标应用获得数据结果
- 通过应用权限进行保护，并可选择使用相同的密钥进行签名来增加额外的安全性

实现简单，通过应用权限和密钥签名保护。但是只能单向通信。



使用 intent：可以通过在 intent 设置额外内容并调用 startActivity 方法将数据从一个 Activity 传递到另一个 Activity。如果正在初始化的 Activity 在其清单中具有“process”属性，则该 Activity 将在自己的进程中启动，但可以访问在 intent 的额外内容中传递给它的数据。

```java
Intent intent = new Intent(this, SomeActivity.class);
intent.putExtra("key", "some serialisable data");
startActivity(intent); //or startActivityForResult(intent,Code) if we expect the invoked activity to response with some data
```

可以使用 intent-filters/Broadcast receivers 来拦截 intent。



# Dalvik 和 ART

Java 的工作原理：为了执行代码，需要 JVM（Java 虚拟机）来执行编译的字节码，并将其转换为机器代码。

Dalvik 遵循相同的理念：Dalvik VM 运行一种不同类型的字节码，称为 DEX（Dalvik 可执行文件），该字节码针对效率进行了更多优化，以便在低性能硬件上更快地运行。它是一个实时 （JIT） 编译器，代码在需要执行时会动态编译。

Android RunTime （ART） 用于相同的目的：将字节码转换为机器代码并执行它。它使用的不是 JIT 编译 ，它使用 Ahead Of Time（AOT） 在安装 APK 或设备空闲时将整个 DEX 转换为机器代码 （dex2oat）。这意味着在执行时速度要快得多，但也需要更多的物理空间。

Dalvik 是 ART 的前身。ART 已在 Android 4.4 中引入，并开始从 Android 7.0 开始使用 AOT 和 JIT 的混合组合，开始遵循不同的编译方法，合成：

1. 应用程序运行的前几次，应用通过 JIT 编译执行。
2. 当设备处于空闲或充电状态时，守护程序会根据 First Run 生成的配置文件编译，使用 AOT 对常用代码执行编译。

可以在 /data/dalvik-cache/profiles/ 中找到每个已安装应用程序的这些配置文件。



# Android Framework

开发者通过 Framework 只需几行代码即可访问复杂的功能。这些代码以 com.android.* 开头的包中提供。这些软件包可以用于不同的范围，例如位置和应用支持（android.location 和 android.app）、网络 （android.net），以及 android.os 的 IPC 支持和核心操作系统服务。

从安全角度来看，这是一个很高的优势。通常，开发人员不必为 native 语言烦恼（避免常见的内存损坏问题），而是使用经过良好测试的代码，当他们需要执行高级或低级功能（例如访问硬件外围设备）时，他们可以保持高级内存安全语言。

```java
import android.net.Wifi
// Get an handle to the WiFi Service
[..]
WifiManager Wifi_manager = (WifiManager) GetApplicationContext().getSystemService(Context.WIFI_SERVICE);
// Get the WifiState
Wifi_manager.getWifiState();
[...]
```

上面是 WiFi 组件交互的简单示例，通过两行代码，完成了：

1. 获取 WiFi 服务的句柄。getSystemService（） 的返回结果是一个泛型 Object（服务的句柄），需要根据所需的服务进行强制转换。
2. 从检索到的管理器中，可以直接调用所需的函数，该函数将执行 IPC 并返回结果。

Android 抽象 service 交互的方式，简化应用程序代码来增强安全性。

有时由于性能原因，有必要在应用程序内运行 native 代码。这是使用 JNI 执行的，它允许在应用程序上下文中调用共享库中的 native 函数。这对于 messaging 应用程序很常见（whatsapp 使用 PJSP（一个 C 库）进行视频会议）。



# Java Native Interface

有时有必要使用标准应用程序中的 C/C++ 等原生代码。这是允许使用 JNI（Java 本机接口）的，它允许 Java 调用 native 函数而没有太大的差异。native 代码在 lib/ 文件夹（的 APK）内的共享库中。 

安全角度看，native 函数可能包含内存损坏问题。



# Binder

Binder 是一个用 C++ 编写的内核模块，主要负责使用客户端-服务器架构让进程安全、透明、轻松地相互通信。进程如何交互的简单性非常糟糕，客户端应用程序只需要调用服务（即客户端体系结构中的服务器）提供的方法，而两者之间的所有内容都由 Binder 处理。“介于两者之间的一切”是指位置、交付和凭据。

当客户端需要与服务通信时，他需要找到目标服务（即目标进程）。Binder 负责定位服务、处理通信、传递消息并检查调用方权限（凭据）。



