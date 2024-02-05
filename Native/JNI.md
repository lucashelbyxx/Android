[JNI Tips](https://developer.android.com/training/articles/perf-jni?hl=zh-cn)



# JNI

[Java 原生接口规范](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)，了解 JNI 的工作原理以及提供的功能。

Java Native Interface，定义了使用 Java 或 Kotlin 编程语言编写）编译的字节码与原生代码（使用 C/C++ 编写）进行交互的方法。



## JNI tips

如需浏览全局 JNI 引用并查看创建和删除全局 JNI 引用的位置，使用 [内存分析器](https://developer.android.com/studio/profile/memory-profiler?hl=zh-cn#jni-references) 中的 **JNI 堆**视图。



### General tips

最小化 JNI 层的占用空间。

- 减少跨 JNI 层编组资源的次数。设计一个接口，最大限度地减少需要编组的数据量以及必须进行数据编组的频率。
- 避免在使用受管理编程语言编写的代码与使用 C++ 编写的代码之间进行异步通信。可使 JNI 接口更易于维护。通常，您可以通过使异步更新使用与界面相同的语言来简化异步界面更新。例如，最好在使用 Java 的两个线程之间执行回调，其中一个线程执行阻塞 C++ 调用，然后在阻塞调用完成时通知界面线程，而不是通过 JNI 从 Java 代码中的界面线程调用 C++ 函数。
- 减少需要访问 JNI 或被 JNI访问的线程数。确实需要以 Java 和 C++ 这两种语言使用线程池，请尝试在池所有者之间（而不是各个工作线程之间）保持 JNI 通信。
- 将接口代码保存在少量易于识别的 C++ 和 Java 源位置，以便于未来的重构。



### JavaVM 和 JNIEnv



### Threads

最好使用 `Thread.start()` 创建需要调用 Java 代码的任何线程。



### jclass、jmethodID 和 jfieldID

通过原生代码访问对象的字段：

- 使用 `FindClass` 获取类的类对象引用
- 使用 `GetFieldID` 获取字段的字段 ID
- 使用适当内容获取字段的内容，例如 `GetIntField`

如需调用方法，首先要获取类对象引用，然后获取方法 ID。ID 通常只是指向内部运行时数据结构的指针。查找它们可能需要进行多次字符串比较，但获得它们后，您可以很快进行实际调用以获取字段或调用相应方法。

性能，建议查找一次值并将结果缓存在原生代码中。由于每个进程只能有一个 JavaVM，因此最好将此类数据存储在静态本地结构中。

在取消加载类之前，类引用、字段 ID 和方法 ID 保证有效。只有当与 ClassLoader 关联的所有类均可进行垃圾回收时，系统才会取消加载类，这种情况很少见，但在 Android 中并非不可能。但请注意，`jclass` 是类引用，必须通过调用 `NewGlobalRef` **来保护**（请参阅下一部分）。

如果您想在加载某个类时缓存 ID，并在取消加载和重新加载该类后自动重新缓存这些 ID，初始化 ID 的正确方法是将与以下类似的代码添加到相应类中：

```java
/*
 * We use a class initializer to allow the native code to cache some
 * field offsets. This native function looks up and caches interesting
 * class/field/method IDs. Throws on failure.
 */
private static native void nativeInit();

static {
    nativeInit();
}
```

在执行 ID 查找的 C/C++ 代码中创建 `nativeClassInit` 方法。该代码将在类初始化时执行一次。如果取消加载该类后又重新加载，该类将再次执行。



### 局部引用和全局引用

传递给原生方法的每个参数，以及 JNI 函数返回几乎每个对象都是“局部引用”。这意味着，它在当前线程中的当前原生方法的持续时间内有效。**在原生方法返回后，即使对象本身继续存在，该引用也无效。**

获取非局部引用的唯一方法是通过 `NewGlobalRef` 和 `NewWeakGlobalRef` 函数。

如果想长时间保留某个引用，则必须使用“全局”引用。`NewGlobalRef` 函数将局部引用作为参数，然后返回全局引用。在调用 `DeleteGlobalRef` 之前，全局引用保证有效。

在缓存从 `FindClass` 返回的 jclass 时，通常使用此模式，例如：

```java
jclass localClass = env->FindClass("MyClass");
jclass globalClass = reinterpret_cast<jclass>(env->NewGlobalRef(localClass));
```

所有 JNI 方法都接受局部引用和全局引用作为参数。对同一对象的引用可能具有不同的值。例如，对同一对象连续调用 `NewGlobalRef` 所返回的值可能有所不同。**如需查看两个引用是否引用同一对象，必须使用 `IsSameObject` 函数。**切勿在原生代码中使用 `==` 比较引用。

这样做的一个后果就是，**不得假定对象引用在原生代码中是常量或唯一**。每次调用方法时，表示对象的值可能不同，并且在连续调用时，两个不同的对象可能具有相同的值。请勿将 `jobject` 值用作键。

“不过度分配”局部引用。实际上，这意味着如果您要创建大量局部引用（可能是在运行对象数组时），应该使用 `DeleteLocalRef` 手动释放它们，而不是让 JNI 为您执行此操作。

注意，`jfieldID` 和 `jmethodID` 是不透明类型，不是对象引用，并且不应传递给 `NewGlobalRef`。函数返回的原始数据指针（如 `GetStringUTFChars` 和 `GetByteArrayElements`）也不属于对象。（它们可以在线程之间传递，并且在匹配的 Release 调用之前一直有效。）

还有一种不寻常的情况值得单独提及。如果您使用 `AttachCurrentThread` 附加原生线程，那么在线程分离之前，您运行的代码绝不会自动释放局部引用。您创建的任何本地引用都必须手动删除。一般来说，在循环中创建局部引用的任何原生代码可能需要执行一些手动删除操作。

请谨慎使用全局引用。全局引用不可避免，但它们难以调试，并且可能会导致难以诊断的内存（不良）行为。



### UTF-8 和 UTF-16



### Primitive arrays

为了在不限制虚拟机实现的情况下使接口尽可能高效，`Get<PrimitiveType>ArrayElements` 系列调用允许运行时返回指向实际元素的指针，或分配一些内存并制作副本。无论采用哪种方式，返回的原始指针保证有效，直到发出相应的 `Release` 调用（这意味着，如果未复制数据，数组对象将被固定，无法在压缩堆期间重新定位）。**您必须 `Release` 自己 `Get` 的每个数组。**此外，如果 `Get` 调用失败，您必须确保自己的代码稍后不会尝试 `Release` NULL 指针。



### 区域调用

如果只复制数据，使用 `Get<Type>ArrayElements` 和 `GetStringChars` 等调用替代方法

```java
jbyte* data = env->GetByteArrayElements(array, NULL);
if (data != NULL) {
    memcpy(buffer, data, len);
    env->ReleaseByteArrayElements(array, data, JNI_ABORT);
}
```

这会获取数组、将第一个 `len` 字节元素复制出其中，然后释放数组。`Get` 调用会固定或复制数组内容，具体取决于实现情况。 代码复制数据（可能是第二次），然后调用 `Release`；在这种情况下，`JNI_ABORT` 确保不会有机会进行第三次复制。

更简单的方式完成相同操作：

```cpp
    env->GetByteArrayRegion(array, 0, len, buffer);
```

这种做法优势：

- 需要一个 JNI 调用而不是两个，从而减少开销。
- 不需要固定或额外复制数据。
- 降低程序员出错的风险 - 不存在操作失败后忘记调用 `Release` 的风险。

同样，您可以使用 `Set<Type>ArrayRegion` 调用将数据复制到数组中，使用 `GetStringRegion` 或 `GetStringUTFRegion` 将字符从 `String` 中复制出来。



### 异常

**在异常挂起时，不得调用大多数 JNI 函数。**

在异常挂起时，您只能调用以下 JNI 函数：

- `DeleteGlobalRef`
- `DeleteLocalRef`
- `DeleteWeakGlobalRef`
- `ExceptionCheck`
- `ExceptionClear`
- `ExceptionDescribe`
- `ExceptionOccurred`
- `MonitorExit`
- `PopLocalFrame`
- `PushLocalFrame`
- `Release<PrimitiveType>ArrayElements`
- `ReleasePrimitiveArrayCritical`
- `ReleaseStringChars`
- `ReleaseStringCritical`
- `ReleaseStringUTFChars`

许多 JNI 调用都会抛出异常，但通常会提供一种更简单的失败检查方法。例如，如果 `NewString` 返回非 NULL 值，则无需检查异常。不过，如果您调用方法（使用 `CallObjectMethod` 等函数），则必须始终检查异常，因为如果系统抛出异常，返回值将无效。

请注意，受管理代码抛出的异常不会展开原生堆栈帧。（Android 不建议使用 C++ 异常，切勿跨越从 C++ 代码到托管代码的 JNI 转换边界抛出 C++ 异常。）JNI `Throw` 和 `ThrowNew` 指令只是在当前线程中设置了异常指针。从原生代码返回到受管理代码后，系统会记录异常并进行相应处理。

原生代码可以通过调用 `ExceptionCheck` 或 `ExceptionOccurred` 来“捕获”异常，然后使用 `ExceptionClear` 将其清除。如果不处理异常就舍弃异常可能会导致问题。

没有用于操控 `Throwable` 对象本身的内置函数，因此，如果您想（比方说）获取异常字符串，就需要找到 `Throwable` 类，查找 `getMessage "()Ljava/lang/String;"` 的方法 ID，然后进行调用，如果结果为非 NULL，请使用 `GetStringUTFChars` 获取您可以传递给 `printf(3)` 或等效项的内容。



### 扩展检查

JNI 很少进行错误检查。错误通常会导致崩溃。Android 还提供了一种名为 CheckJNI 的模式，其中 JavaVM 和 JNIEnv 函数表指针已切换为在调用标准实现之前执行一系列扩展的检查的函数表。

额外检查包括：

- 数组：尝试分配大小为负值的数组。
- 错误指针：将错误的 jarray/jclass/jobject/jstring 传递给 JNI 调用，或者将 NULL 指针传递给带有不可设为 null 的参数的 JNI 调用。
- 类名称：将类名称的“java/lang/String”样式之外的所有内容传递给 JNI 调用。
- 关键调用：在“关键”get 及其相应 release 之间调用 JNI。
- 直接字节缓冲区：将错误参数传递给 `NewDirectByteBuffer`。
- 异常：在异常挂起时调用 JNI。
- JNIEnv*：使用错误线程中的 JNIEnv*。
- jfieldID：使用 NULL jfieldID，或者使用 jfieldID 将字段设置为错误类型的值（例如，尝试将 StringBuilder 分配给 String 字段），或者使用静态字段的 jfieldID 设置实例字段（反之亦然），或者将一个类中的 jfieldID 与另一个类的实例搭配使用。
- jmethodID：在调用 `Call*Method` JNI 时使用错误类型的 jmethodID：返回类型不正确、静态/非静态不匹配、“this”类型错误（对于非静态调用）或类错误（对于静态调用）。
- 引用：对错误类型的引用使用 `DeleteGlobalRef`/`DeleteLocalRef`。
- Release 模式：将错误的 release 模式传递给 release 调用（除 `0`、`JNI_ABORT` 或 `JNI_COMMIT` 之外的内容）。
- 类型安全：从原生方法返回不兼容的类型（例如，从声明返回 String 的方法返回 StringBuilder）。
- UTF-8：将无效的[修改后的 UTF-8](https://en.wikipedia.org/wiki/UTF-8#Modified_UTF-8) 字节序列传递给 JNI 调用。



### 原生库

可以使用标准 [`System.loadLibrary`](https://developer.android.com/reference/java/lang/System?hl=zh-cn#loadLibrary(java.lang.String)) 从共享库加载原生代码。

从静态类初始化程序中调用 `System.loadLibrary`（或 `ReLinker.loadLibrary`）。该参数是“未修饰”的库名称，因此要加载 `libfubar.so`，您需要传入 `"fubar"`。

如果只有一个类具有原生方法，则合理做法是，在该类的静态初始化程序中调用 `System.loadLibrary`。否则，您可能需要从 `Application` 进行调用，这样才能知道始终会加载该库，而且总是会提前加载。

运行时可以通过两种方式找到您的原生方法。可以使用 `RegisterNatives` 明确注册它们，也可以让运行时使用 `dlsym` 动态查找它们。`RegisterNatives` 的优势在于，您可以预先检查符号是否存在，而且可以通过不导出 `JNI_OnLoad` 以外的任何其他内容来获得更小且更快的共享库。让运行时发现函数的优势在于，要编写的代码会稍微少一些。

如需使用 `RegisterNatives`，请执行以下操作：

- 提供 `JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved)` 函数。
- 在 `JNI_OnLoad` 中，使用 `RegisterNatives` 注册所有原生方法。
- 使用 `-fvisibility=hidden` 进行构建，以便从您的库中仅导出 `JNI_OnLoad`。这样可以生成更快、更小的代码，并避免与加载到应用的其他库发生潜在冲突（但如果您的应用在原生代码中崩溃，创建的堆栈轨迹用处不大）。

静态初始化程序应如下所示：

[Kotlin](https://developer.android.com/training/articles/perf-jni?hl=zh-cn#kotlin)[Java](https://developer.android.com/training/articles/perf-jni?hl=zh-cn#java)

```java
static {
    System.loadLibrary("fubar");
}
```

如果使用 C++ 编写，`JNI_OnLoad` 函数应如下所示：

```cpp
JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
        return JNI_ERR;
    }

    // Find your class. JNI_OnLoad is called from the correct class loader context for this to work.
    jclass c = env->FindClass("com/example/app/package/MyClass");
    if (c == nullptr) return JNI_ERR;

    // Register your class' native methods.
    static const JNINativeMethod methods[] = {
        {"nativeFoo", "()V", reinterpret_cast<void*>(nativeFoo)},
        {"nativeBar", "(Ljava/lang/String;I)Z", reinterpret_cast<void*>(nativeBar)},
    };
    int rc = env->RegisterNatives(c, methods, sizeof(methods)/sizeof(JNINativeMethod));
    if (rc != JNI_OK) return rc;

    return JNI_VERSION_1_6;
}
```

如需改为使用“发现”原生方法，您需要以特定方式为这些方法命名（如需了解详情，请参阅 [JNI 规范](http://java.sun.com/javase/6/docs/technotes/guides/jni/spec/design.html#wp615)）。这意味着，如果方法签名有误，您在第一次实际调用该方法时才会知道。

从 `JNI_OnLoad` 进行的任何 `FindClass` 调用都将在用于加载共享库的类加载器的上下文中解析类。从其他上下文调用时，`FindClass` 会使用与 Java 堆栈顶部的方法关联的类加载器；如果没有（因为调用来自刚刚附加的原生线程），它会使用“系统”类加载器。系统类加载器不知道应用的类，因此您无法在该上下文中使用 `FindClass` 查找您自己的类。这使得 `JNI_OnLoad` 成为查找和缓存类的便捷位置：拥有有效的 `jclass` [全局引用](https://developer.android.com/training/articles/perf-jni?hl=zh-cn#local_and_global_references)后，您就可以从任何附加的线程使用它。



### 使用 `@FastNative` 和 `@CriticalNative` 加快原生调用速度



### 64 位注意事项





## 常见问题

### UnsatisfiedLinkError

### FindClass 找不到我的类

### 使用原生代码共享原始数据

从受管理代码和原生代码访问大型原始数据缓冲区。常见示例包括操纵位图或声音样本。有两种基本方法。

您可以将数据存储在 `byte[]` 中。这样就可以从受管理代码进行非常快速访问。但在原生端，无法保证您能够在不复制数据的情况下访问数据。在某些实现中，`GetByteArrayElements` 和 `GetPrimitiveArrayCritical` 将返回指向托管堆中原始数据的实际指针，但在其他实现中，它会在原生堆上分配缓冲区并复制数据。

另一种方法是将数据存储在直接字节缓冲区中。这类凭据可以使用 `java.nio.ByteBuffer.allocateDirect` 或 JNI `NewDirectByteBuffer` 函数创建。与常规字节缓冲区不同，存储空间不在托管堆上分配，并且始终可以直接从原生代码访问（使用 `GetDirectBufferAddress` 获取地址）。根据直接字节缓冲区访问的实现方式，通过托管代码访问数据可能会很慢。

选择使用哪种方法取决于以下两个因素：

1. 大部分数据访问是否通过使用 Java 或 C/C++ 编写的代码进行？
2. 如果数据最终被传递到系统 API，它必须以什么形式呈现？（例如，如果数据最终传递给采用 byte[] 的函数，则在直接 `ByteBuffer` 中进行处理可能是不明智的。）

如果这两种方法不分伯仲，请使用直接字节缓冲区。对它们的支持直接内置于 JNI 中，并且在未来的版本中，性能应该会得到提升。