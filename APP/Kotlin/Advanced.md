todo

[Exceptions in coroutines](https://medium.com/androiddevelopers/exceptions-in-coroutines-ce8da1ec060c)

[Cancellation in coroutines. ](https://medium.com/androiddevelopers/cancellation-in-coroutines-aa6b90163629)

[Coroutines & Patterns for work that shouldn’t be cancelled ](https://medium.com/androiddevelopers/coroutines-patterns-for-work-that-shouldnt-be-cancelled-e26c40f142ad)

[有关 Kotlin 协程和流程的其他资源  | Android Developers](https://developer.android.com/kotlin/coroutines/additional-resources?hl=zh-cn)



# 协程

协程有助于管理长时间运行的任务，否则这些任务可能会阻塞主线程并导致应用无响应。



## Features

- 轻量级：支持挂起，可以在单个线程上运行多个协程，不会阻塞运行协程的线程。suspend 比 block 节省内存，而且支持许多并发操作。
- 减少内存泄漏：使用 [*structured concurrency*](https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency) 在作用域内执行操作
- 内置 cancellation 支持： [Cancellation](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html) 通过正常运行的协程层次自动传播



## 依赖项

```
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
}
```



## 后台线程执行

```kotlin
sealed class Result<out R> {
    data class Success<out T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
}

class LoginRepository(private val responseParser: LoginResponseParser) {
    private const val loginUrl = "https://example.com/login"

    // Function that makes the network request, blocking the current thread
    fun makeLoginRequest(
        jsonBody: String
    ): Result<LoginResponse> {
        val url = URL(loginUrl)
        (url.openConnection() as? HttpURLConnection)?.run {
            requestMethod = "POST"
            setRequestProperty("Content-Type", "application/json; utf-8")
            setRequestProperty("Accept", "application/json")
            doOutput = true
            outputStream.write(jsonBody.toByteArray())
            return Result.Success(responseParser.parse(inputStream))
        }
        return Result.Error(Exception("Cannot open HttpURLConnection"))
    }
}

class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {
        val jsonBody = "{ username: \"$username\", token: \"$token\"}"
        loginRepository.makeLoginRequest(jsonBody)
    }
}	
```

makeLoginRequest 是同步的，并阻塞调用线程。用户点击按钮时，ViewModel 会触发网络请求，阻塞 UI 线程。

将操作移出主线程最简单的方案是创建一个新的协程并在 I/O 线程上执行网络请求。

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {
        // Create a new coroutine to move the execution off the UI thread
        viewModelScope.launch(Dispatchers.IO) {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            loginRepository.makeLoginRequest(jsonBody)
        }
    }
}
```

- viewModelScope 是 ViewModel KTX 扩展里的预定义 CoroutineScope。所有的协程都必须在作用域 scope 中运行。一个 CoroutineScope 管理多个相关的协程。
- launch 函数创建一个协程，并将函数体的执行分派给相应的调度程序。
- Dispatchers.IO 表明此协程在 IO 操作保留的线程上执行。



login 函数执行如下：

- 主线程的 View 层调用 login 函数
- launch 创建一个新的协程，并在 IO 操作的线程上独立发出网络请求
- 协程运行时，login 函数可能在网络请求完成之前会继续执行并返回



协程以 viewModelScope 启动，因此它在 ViewModel 的范围内执行。如果 ViewModel 由于用户离开而被 destroy，viewModelScope 会自动取消，其中所有运行的协程也会取消。

这里每次调用 makeLoginRequest 都需要将执行显式移出主线程。下面修改 Repository 可以解决这个问题。



## 协程实现主线程安全

函数不阻塞主线程上的 UI 更新时，它就是主线程安全的。makeLoginRequest 函数不是主线程安全的，因为从主线程调用 makeLoginRequest 会阻塞 UI。使用协程库中的 withContext() 函数将协程的执行移动到其他线程。

```kotlin
class LoginRepository(...) {
    ...
    suspend fun makeLoginRequest(
        jsonBody: String
    ): Result<LoginResponse> {

        // Move the execution of the coroutine to the I/O dispatcher
        return withContext(Dispatchers.IO) {
            // Blocking network request code
        }
    }
}
```

withContext(Dispatchers.IO) 将协程的执行移到 IO 线程，使调用函数成为主线程安全函数。

suspend 关键字表示 Kotlin 强制从协程中调用函数的方法。



下面示例中，协程是在 LoginViewModel 中创建的。当 makeLoginRequest 的执行移出主线程时，login 函数的协程现在可以在主线程中执行。

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {

        // Create a new coroutine on the UI thread
        viewModelScope.launch {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"

            // Make the network call and suspend execution until it finishes
            val result = loginRepository.makeLoginRequest(jsonBody)

            // Display result of the network request to the user
            when (result) {
                is Result.Success<LoginResponse> -> // Happy path
                else -> // Show error in UI
            }
        }
    }
}
```

与前面 login 的不同：launch 不使用 Dispatcher.IO 参数，当 launch 不带 Dispatcher 参数时，任何从 viewModelScope 启动的协程都会在主线程运行

login 函数执行：

- 主线程的 View 层调用 login 函数
- launch 在主线程上创建一个新的协程，然后该协程开始执行
- 在协程中，对 loginRepository.makeLoginRequest 的调用会暂停协程的继续执行，知道 makeLoginRequest 中的 withContext 块运行完成
- withContext 块完成后，login 中的协程将使用网络请求的结果在主线程上恢复执行



## 异常处理

要处理 Repository 层可能引发的异常，可以使用 Kotlin 内置的异常支持。

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun makeLoginRequest(username: String, token: String) {
        viewModelScope.launch {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            val result = try {
                loginRepository.makeLoginRequest(jsonBody)
            } catch(e: Exception) {
                Result.Error(Exception("Network request failed"))
            }
            when (result) {
                is Result.Success<LoginResponse> -> // Happy path
                else -> // Show error in UI
            }
        }
    }
}
```

示例中，任何 makeLoginRequest 调用引发的异常都作为 error 处理。



# 协程高级概念

## 管理长时间运行的任务

协程在常规函数基础上添加了两个操作来处理长时间运行的任务，除了 invoke (or call) 和 return 之外，协程还添加了 suspend 和 resume：

- suspend 暂停当前协程的执行，保存所有局部变量
- resume 继续从暂停的地方执行暂停的协程

只能从其他 suspend 函数或者使用 coroutine builder（例如 launch 启动新的协程）来调用 suspend 函数。

长时间运行任务的简单协程实现：

```kotlin
suspend fun fetchDocs() {                             // Dispatchers.Main
    val result = get("https://developer.android.com") // Dispatchers.IO for `get`
    show(result)                                      // Dispatchers.Main
}

suspend fun get(url: String) = withContext(Dispatchers.IO) { /* ... */ }
```

示例中，get() 仍然在主线程上运行，但它在开始网络请求之前挂起协程，当网络请求完成时，`get` 会恢复已挂起的协程，而不是使用回调通知主线程。

Kotlin 使用堆栈帧管理要运行哪个函数以及所有局部变量。挂起协程时，系统会复制并保存当前的堆栈帧以供稍后使用。恢复时，会将堆栈帧从其保存位置复制回来，然后函数再次开始运行。即使代码可能看起来像普通的顺序阻塞请求，协程也能确保网络请求避免阻塞主线程。



## 使用协程确保主线程安全

Kotlin 协程使用 Dispatchers（调度、分发）确定哪些线程用于执行协程。要在主线程之外运行代码，可以让 Kotlin 协程在 Default 或 IO 调度程序上执行工作。在 Kotlin 中，所有协程都必须在调度程序中运行，即使它们在主线程上运行也是如此。协程可以自行挂起，而调度程序负责将其恢复。

Kotlin 提供了三个调度程序，以用于指定应在何处运行协程：

- **Dispatchers.Main** - 使用此调度程序可在 Android 主线程上运行协程。此调度程序只能用于与界面交互和执行快速工作。示例包括调用 `suspend` 函数，运行 Android 界面框架操作，以及更新 [`LiveData`](https://developer.android.com/topic/libraries/architecture/livedata?hl=zh-cn) 对象。
- **Dispatchers.IO** - 此调度程序经过了专门优化，适合在主线程之外执行磁盘或网络 I/O。示例包括使用 [Room 组件](https://developer.android.com/topic/libraries/architecture/room?hl=zh-cn)、从文件中读取数据或向文件中写入数据，以及运行任何网络操作。
- **Dispatchers.Default** - 此调度程序经过了专门优化，适合在主线程之外执行占用大量 CPU 资源的工作。用例示例包括对列表排序和解析 JSON。

可以使用调度程序重新定义 `get` 函数。在 `get` 的主体内，调用 `withContext(Dispatchers.IO)` 来创建一个在 IO 线程池中运行的块。您放在该块内的任何代码都始终通过 `IO` 调度程序执行。由于 `withContext` 本身就是一个挂起函数，因此函数 `get` 也是一个挂起函数。

```kotlin
suspend fun fetchDocs() {                      // Dispatchers.Main
    val result = get("developer.android.com")  // Dispatchers.Main
    show(result)                               // Dispatchers.Main
}

suspend fun get(url: String) =                 // Dispatchers.Main
    withContext(Dispatchers.IO) {              // Dispatchers.IO (main-safety block)
        /* perform network IO here */          // Dispatchers.IO (main-safety block)
    }                                          // Dispatchers.Main
}
```

借助协程，您可以通过精细控制来调度线程。由于 `withContext()` 可让您在不引入回调的情况下控制任何代码行的线程池，因此您可以将其应用于非常小的函数，例如从数据库中读取数据或执行网络请求。一种不错的做法是使用 `withContext()` 来确保每个函数都是主线程安全的，这意味着，您可以从主线程调用每个函数。这样，调用方就从不需要考虑应该使用哪个线程来执行函数了。

示例中，`fetchDocs()` 在主线程上执行；不过，它可以安全地调用 `get`，这样会在后台执行网络请求。由于协程支持 `suspend` 和 `resume`，因此 `withContext` 块完成后，主线程上的协程会立即根据 `get` 结果恢复。



### **withContext 的性能**

与基于回调的实现相比，[`withContext()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html) 不会增加额外的开销。此外，还可以优化 `withContext()` 调用，使其超越基于回调的实现。例如，如果某个函数对一个网络进行十次调用，您可以使用外部 `withContext()` 让 Kotlin 只切换一次线程。这样，即使网络库多次使用 `withContext()`，它也会留在同一调度程序上，并避免切换线程。此外，Kotlin 还优化了 `Dispatchers.Default` 与 `Dispatchers.IO` 之间的切换，以尽可能避免线程切换。

但是，利用一个使用线程池的调度程序（例如 `Dispatchers.IO` 或 `Dispatchers.Default`）不能保证块在同一线程上从上到下执行。在某些情况下，Kotlin 协程在 `suspend` 和 `resume` 后可能会将执行工作移交给另一个线程。这意味着，对于整个 `withContext()` 块，线程局部变量可能并不指向同一个值。



## 启动协程

两种方式来启动协程：

- [`launch`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 会启动新协程而不将结果返回给调用方

- [`async`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 可启动一个新协程，并允许您使用一个名为 `await` 的挂起函数返回结果。

通常，应使用 `launch` 从常规函数启动新协程，因为常规函数无法调用 `await`。只有在另一个协程内或在挂起函数内且在执行并行分解时，才使用 `async`。



### **并行分解**

在 `suspend` 函数启动的所有协程都必须在该函数返回结果时停止，因此您可能需要保证这些协程在返回结果之前完成。借助 Kotlin 中的结构化并发机制，您可以定义用于启动一个或多个协程的 `coroutineScope`。然后，您可以使用 `await()`（针对单个协程）或 `awaitAll()`（针对多个协程）保证这些协程在从函数返回结果之前完成。

例如，假设我们定义一个用于异步获取两个文档的 `coroutineScope`。通过对每个延迟引用调用 `await()`，我们可以保证这两项 `async` 操作在返回值之前完成：

```kotlin
suspend fun fetchTwoDocs() =
    coroutineScope {
        val deferredOne = async { fetchDoc(1) }
        val deferredTwo = async { fetchDoc(2) }
        deferredOne.await()
        deferredTwo.await()
    }
```

您还可以对集合使用 `awaitAll()`，如以下示例所示：

```kotlin
suspend fun fetchTwoDocs() =        // called on any Dispatcher (any thread, possibly Main)
    coroutineScope {
        val deferreds = listOf(     // fetch two docs at the same time
            async { fetchDoc(1) },  // async returns a result for the first doc
            async { fetchDoc(2) }   // async returns a result for the second doc
        )
        deferreds.awaitAll()        // use awaitAll to wait for both network requests
    }
```

虽然 `fetchTwoDocs()` 使用 `async` 启动新协程，但该函数使用 `awaitAll()` 等待启动的协程完成后才会返回结果。不过请注意，即使我们没有调用 `awaitAll()`，`coroutineScope` 构建器也会等到所有新协程都完成后才恢复名为 `fetchTwoDocs` 的协程。

此外，`coroutineScope` 会捕获协程抛出的所有异常，并将其传送回调用方。

细了解并行分解，请参阅[编写挂起函数](https://kotlinlang.org/docs/reference/coroutines/composing-suspending-functions.html)



## 协程概念

### CoroutineScope

[`CoroutineScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) 使用 `launch` 或 `async` 跟踪它创建的任何协程。可以随时调用 `scope.cancel()` 以取消正在进行的工作（即正在运行的协程）。在 Android 中，某些 KTX 库为某些生命周期类提供自己的 `CoroutineScope`。例如，`ViewModel` 有 [`viewModelScope`](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary?hl=zh-cn#(androidx.lifecycle.ViewModel).viewModelScope:kotlinx.coroutines.CoroutineScope)，`Lifecycle` 有 [`lifecycleScope`](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary?hl=zh-cn#lifecyclescope)。不过，与调度程序不同，`CoroutineScope` 不运行协程。

如果您需要创建自己的 `CoroutineScope` 以控制协程在应用的特定层中的生命周期，则可以创建一个如下所示的 CoroutineScope：

```kotlin
class ExampleClass {

    // Job and Dispatcher are combined into a CoroutineContext which
    // will be discussed shortly
    val scope = CoroutineScope(Job() + Dispatchers.Main)

    fun exampleMethod() {
        // Starts a new coroutine within the scope
        scope.launch {
            // New coroutine that can call suspend functions
            fetchDocs()
        }
    }

    fun cleanUp() {
        // Cancel the scope to cancel ongoing coroutines work
        scope.cancel()
    }
}
```

已取消的作用域无法再创建协程。因此，仅当控制其生命周期的类被销毁时，才应调用 `scope.cancel()`。使用 `viewModelScope` 时，[`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn) 类会在 ViewModel 的 `onCleared()` 方法中自动为您取消作用域。

### Job

[`Job`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) 是协程的句柄。使用 `launch` 或 `async` 创建的每个协程都会返回一个 `Job` 实例，该实例是相应协程的唯一标识并管理其生命周期。您还可以将 `Job` 传递给 `CoroutineScope` 以进一步管理其生命周期，如以下示例所示：

```kotlin
class ExampleClass {
    ...
    fun exampleMethod() {
        // Handle to the coroutine, you can control its lifecycle
        val job = scope.launch {
            // New coroutine
        }

        if (...) {
            // Cancel the coroutine started above, this doesn't affect the scope
            // this coroutine was launched in
            job.cancel()
        }
    }
}
```

### CoroutineContext

[`CoroutineContext`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/index.html) 使用以下元素集定义协程的行为：

- [`Job`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)：控制协程的生命周期。
- [`CoroutineDispatcher`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html)：将工作分派到适当的线程。
- [`CoroutineName`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-name/index.html)：协程的名称，可用于调试。
- [`CoroutineExceptionHandler`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html)：处理未捕获的异常。

对于在作用域内创建的新协程，系统会为新协程分配一个新的 `Job` 实例，而从包含作用域继承其他 `CoroutineContext` 元素。可以通过向 `launch` 或 `async` 函数传递新的 `CoroutineContext` 替换继承的元素。请注意，将 `Job` 传递给 `launch` 或 `async` 不会产生任何效果，因为系统始终会向新协程分配 `Job` 的新实例。

```kotlin
class ExampleClass {
    val scope = CoroutineScope(Job() + Dispatchers.Main)

    fun exampleMethod() {
        // Starts a new coroutine on Dispatchers.Main as it's the scope's default
        val job1 = scope.launch {
            // New coroutine with CoroutineName = "coroutine" (default)
        }

        // Starts a new coroutine on Dispatchers.Default
        val job2 = scope.launch(Dispatchers.Default + CoroutineName("BackgroundCoroutine")) {
            // New coroutine with CoroutineName = "BackgroundCoroutine" (overridden)
        }
    }
}
```

CoroutineExceptionHandler [Exceptions in coroutines.](https://medium.com/androiddevelopers/exceptions-in-coroutines-ce8da1ec060c)



# 测试协程

协程的单元测试代码执行可能是异步的，而且可能发生在多个线程中。



## 在测试中调用 suspend 函数

要在协程中才能调用挂起函数，由于 Junit 测试函数本身不是挂起函数，需要在测试中调用协程构建器来启动新的协程。

runTest 是用于测试的协程构建器，使用此构建器可封装包含协程的任何测试。协程不仅可以直接在测试主体中启动，还可以通过测试中使用的对象启动。

```kotlin
suspend fun fetchData(): String {
    delay(1000L)
    return "Hello world"
}

@Test
fun dataShouldBeHelloWorld() = runTest {
    val data = fetchData()
    assertEquals("Hello world", data)
}
```

要测试基本挂起函数，可以将测试代码封装在 `runTest` 中，这样可自动跳过协程中的任何延迟，从而使上述测试只需不到一秒种的时间即可完成。

不过，根据被测试代码的具体情况，您还需考虑其他事项：

- 除 `runTest` 创建的顶级测试协程之外，如果您的代码还创建了新的协程，则您需要[选择适当的 `TestDispatcher`](https://developer.android.com/kotlin/coroutines/test?hl=zh-cn#testdispatchers)，以控制这些新协程的调度方式。
- 如果您的代码将协程执行移至其他调度程序（例如，通过使用 [`withContext`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)），`runTest` 通常仍可正常运行，但不会再跳过延迟，并且由于代码会在多个线程上运行，测试的可预测性将降低。出于这些原因，您应在测试中[注入测试调度程序](https://developer.android.com/kotlin/coroutines/test?hl=zh-cn#injecting-test-dispatchers)，以替换实际调度程序。



## TestDispatchers

[`TestDispatchers`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-dispatcher/index.html) 是用于测试的 [`CoroutineDispatcher`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html) 实现。如果要在测试期间创建新的协程，您需要使用 `TestDispatchers`，以使新协程的执行可预测。

`TestDispatcher` 有两种可用的实现：[`StandardTestDispatcher`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-standard-test-dispatcher.html) 和 [`UnconfinedTestDispatcher`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-unconfined-test-dispatcher.html)，可分别对新启动的协程执行不同的调度。两者都使用 [`TestCoroutineScheduler`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/index.html) 来控制虚拟时间并管理测试中正在运行的协程。

**一个测试中只能使用一个调度器实例**，且所有 `TestDispatchers` 应共用该调度器。

为了启动顶级测试协程，`runTest` 会创建一个 [`TestScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-scope.html)，它是 [`CoroutineScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) 的实现，将始终使用 `TestDispatcher`。如果未指定，`TestScope` 将默认创建 `StandardTestDispatcher`，并将其用于运行顶级测试协程。

`runTest` 会跟踪在其 `TestScope` 的调度程序所使用的调度器上排队的协程，只要该调度器上还有待处理的工作，它就不会返回。



### **StandardTestDispatcher**

如果新协程是在 `StandardTestDispatcher` 上启动的，则这些协程会在底层调度器上排队，以便在测试线程可供使用时运行。若要让这些新协程运行，您需要“让出”测试线程（将其释放出来，以供其他协程使用）。这种排队行为可让您精确控制测试期间新协程的运行方式，类似于正式版代码中的协程调度。

如果在顶级测试协程执行期间未让出测试线程，那么所有新协程都只会在测试协程完成后（但在 `runTest` 返回之前）运行：

```kotlin
@Test
fun standardTest() = runTest {
    val userRepo = UserRepository()

    launch { userRepo.register("Alice") }
    launch { userRepo.register("Bob") }

    assertEquals(listOf("Alice", "Bob"), userRepo.getAllUsers()) // ❌ Fails
}
StandardTestDispatcherTest.kt
```

您可以通过多种方式让出测试协程，从而使排队的协程运行。所有以下调用都可在返回之前让其他协程在测试线程上运行：

- [`advanceUntilIdle`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/advance-until-idle.html)：在调度器上运行所有其他协程，直到队列中没有任何内容。默认选择，可让所有待处理的协程运行，适用于大多数测试场景。
- [`advanceTimeBy`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/advance-time-by.html)：将虚拟时间提前指定时长，并运行已调度为在该虚拟时间点之前运行的所有协程。
- [`runCurrent`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-scheduler/run-current.html)：运行已调度为在当前虚拟时间运行的协程。

如需修正之前的测试，可使用 `advanceUntilIdle` 先让两个待处理的协程执行其工作，然后再继续执行断言：

```kotlin
@Test
fun standardTest() = runTest {
    val userRepo = UserRepository()

    launch { userRepo.register("Alice") }
    launch { userRepo.register("Bob") }
    advanceUntilIdle() // Yields to perform the registrations

    assertEquals(listOf("Alice", "Bob"), userRepo.getAllUsers()) // ✅ Passes
}
```



### **UnconfinedTestDispatcher**

如果新协程是在 `UnconfinedTestDispatcher` 上启动的，则这些协程会在当前线程上快速启动。也就是说，这些协程会立即开始运行，而不会等到其协程构建器返回之后再运行。在许多情况下，这种调度行为会使测试代码更加简单，因为您无需手动让出测试线程即可让新协程运行。



## 注入测试调度程序

被测试代码可能会使用调度程序来切换线程（使用 [`withContext`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)）或启动新协程。在多个线程上并行执行代码时，测试可能会变得不稳定。如果在您无法控制的后台线程上运行代码，可能就难以在正确的时间执行断言或等待任务完成。

在测试中，请将这些调度程序替换为 `TestDispatchers` 的实例。这样做有几个好处：

- 代码将在单个测试线程上运行，让测试更具确定性
- 您可以控制新协程的调度和执行方式
- TestDispatchers 使用虚拟时间调度器，它可以自动跳过延迟，并允许您手动将时间提前

可以使用[依赖项注入](https://developer.android.com/training/dependency-injection?hl=zh-cn)为类提供调度程序，从而在测试中轻松替换实际调度程序。示例中，将注入一个 `CoroutineDispatcher`，但也可以注入更广泛的 [`CoroutineContext`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/) 类型，从而在测试期间实现更大的灵活性。



## 设置主调度程序

## 测试之外创建调度器

## 创建 TestScope

## 注入作用域



## 

# 协程最佳实践

## 注入调度程序 Inject Dispatchers

在创建新协程或调用 `withContext` 时，请勿对 `Dispatchers` 进行硬编码。

```kotlin
// DO inject Dispatchers
class NewsRepository(
    private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    suspend fun loadNews() = withContext(defaultDispatcher) { /* ... */ }
}

// DO NOT hardcode Dispatchers
class NewsRepository {
    // DO NOT use Dispatchers.Default directly, inject it instead
    suspend fun loadNews() = withContext(Dispatchers.Default) { /* ... */ }
}
```

这种依赖项注入模式可以降低测试难度，因为您可以使用[测试调度程序](https://developer.android.com/kotlin/coroutines/coroutines-best-practices?hl=zh-cn#test-coroutine-dispatcher)替换单元测试和插桩测试中的这些调度程序，以提高测试的确定性。



## 挂起函数在主线程安全调用

挂起函数应该是主线程安全的，这意味着可以安全地从主线程调用挂起函数。如果某个类在协程中执行长期运行的阻塞操作，那么该类负责使用 `withContext` 将执行操作移出主线程。

```kotlin
class NewsRepository(private val ioDispatcher: CoroutineDispatcher) {

    // As this operation is manually retrieving the news from the server
    // using a blocking HttpURLConnection, it needs to move the execution
    // to an IO dispatcher to make it main-safe
    suspend fun fetchLatestNews(): List<Article> {
        withContext(ioDispatcher) { /* ... implementation ... */ }
    }
}

// This use case fetches the latest news and the associated author.
class GetLatestNewsWithAuthorsUseCase(
    private val newsRepository: NewsRepository,
    private val authorsRepository: AuthorsRepository
) {
    // This method doesn't need to worry about moving the execution of the
    // coroutine to a different thread as newsRepository is main-safe.
    // The work done in the coroutine is lightweight as it only creates
    // a list and add elements to it
    suspend operator fun invoke(): List<ArticleWithAuthor> {
        val news = newsRepository.fetchLatestNews()

        val response: List<ArticleWithAuthor> = mutableEmptyList()
        for (article in news) {
            val author = authorsRepository.getAuthor(article.author)
            response.add(ArticleWithAuthor(article, author))
        }
        return Result.Success(response)
    }
}
```

此模式可以提高应用的可伸缩性，因为调用挂起函数的类无需担心使用哪个 `Dispatcher` 来处理哪种类型的工作。该责任将由执行相关工作的类承担。



## ViewModel 创建协程

[`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn) 类应首选创建协程，而不是公开挂起函数来执行业务逻辑。如果只需要发出一个值，而不是使用数据流公开状态，`ViewModel` 中的挂起函数就会非常有用。

```kotlin
// DO create coroutines in the ViewModel
class LatestNewsViewModel(
    private val getLatestNewsWithAuthors: GetLatestNewsWithAuthorsUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow<LatestNewsUiState>(LatestNewsUiState.Loading)
    val uiState: StateFlow<LatestNewsUiState> = _uiState

    fun loadNews() {
        viewModelScope.launch {
            val latestNewsWithAuthors = getLatestNewsWithAuthors()
            _uiState.value = LatestNewsUiState.Success(latestNewsWithAuthors)
        }
    }
}

// Prefer observable state rather than suspend functions from the ViewModel
class LatestNewsViewModel(
    private val getLatestNewsWithAuthors: GetLatestNewsWithAuthorsUseCase
) : ViewModel() {
    // DO NOT do this. News would probably need to be refreshed as well.
    // Instead of exposing a single value with a suspend function, news should
    // be exposed using a stream of data as in the code snippet above.
    suspend fun loadNews() = getLatestNewsWithAuthors()
}
```

视图不应直接触发任何协程来执行业务逻辑，而应将这项工作委托给 `ViewModel`。这样一来，业务逻辑就会变得更易于测试，因为可以对 `ViewModel` 对象进行单元测试，而不必使用测试视图所必需的插桩测试。

此外，如果工作是在 `viewModelScope` 中启动，您的协程将在配置更改后自动保留。如果您改用 `lifecycleScope` 创建协程，则必须手动进行处理该操作。



## 不公开可变类型

向其他类公开不可变类型。这样一来，对可变类型的所有更改都会集中在一个类中，便于在出现问题时进行调试。

```kotlin
// DO expose immutable types
class LatestNewsViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(LatestNewsUiState.Loading)
    val uiState: StateFlow<LatestNewsUiState> = _uiState

    /* ... */
}

class LatestNewsViewModel : ViewModel() {

    // DO NOT expose mutable types
    val uiState = MutableStateFlow(LatestNewsUiState.Loading)

    /* ... */
}
```



## 数据层和业务层公开挂起函数和流

数据层和业务层中的类通常会公开函数以执行一次性调用，或接收数据随时间变化的通知。这些层中的类应该**针对一次性调用公开挂起函数**，并**公开数据流以接收关于数据更改的通知**。

```kotlin
// Classes in the data and business layer expose
// either suspend functions or Flows
class ExampleRepository {
    suspend fun makeNetworkRequest() { /* ... */ }

    fun getExamples(): Flow<Example> { /* ... */ }
}
```

采用该最佳实践后，调用方（通常是presentation层）能够控制这些层中发生的工作的执行和生命周期，并在需要时取消相应工作。

### **在业务层和数据层中创建协程**

对于数据层或业务层中因不同原因而需要创建协程的类，它们可以选择不同的选项。

如果仅当用户查看当前屏幕时，要在这些协程中完成的工作才具有相关性，则应遵循调用方的生命周期。在大多数情况下，调用方是 ViewModel，当用户离开屏幕并且 ViewModel 被清除时，调用将被取消。在这种情况下，应使用 [`coroutineScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 或 [`supervisorScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html)。

```kotlin
class GetAllBooksAndAuthorsUseCase(
    private val booksRepository: BooksRepository,
    private val authorsRepository: AuthorsRepository,
) {
    suspend fun getBookAndAuthors(): BookAndAuthors {
        // In parallel, fetch books and authors and return when both requests
        // complete and the data is ready
        return coroutineScope {
            val books = async { booksRepository.getAllBooks() }
            val authors = async { authorsRepository.getAllAuthors() }
            BookAndAuthors(books.await(), authors.await())
        }
    }
}
```

如果只要应用处于打开状态，要完成的工作就具有相关性，并且此工作不限于特定屏幕，那么此工作的存在时间应该比调用方的生命周期更长。对于这种情况，您应使用外部 `CoroutineScope`。

```kotlin
class ArticlesRepository(
    private val articlesDataSource: ArticlesDataSource,
    private val externalScope: CoroutineScope,
) {
    // As we want to complete bookmarking the article even if the user moves
    // away from the screen, the work is done creating a new coroutine
    // from an external scope
    suspend fun bookmarkArticle(article: Article) {
        externalScope.launch { articlesDataSource.bookmarkArticle(article) }
            .join() // Wait for the coroutine to complete
    }
}
```

`externalScope` 应由存在时间比当前屏幕更长的类进行创建和管理，并且可由 `Application` 类或作用域限定为导航图的 `ViewModel` 进行管理。



## 测试中注入 TestDispatchers

## 避免使用 GlobalScope

通过使用 [`GlobalScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)，您将对类使用的 `CoroutineScope` 进行硬编码，而这会带来一些问题：

- 提高硬编码值。如果您对 `GlobalScope` 进行硬编码，则可能同时对 `Dispatchers` 进行硬编码。
- 这会让测试变得非常困难，因为您的代码是在非受控的作用域内执行的，您将无法控制其执行。
- 您无法设置一个通用的 `CoroutineContext` 来对内置于作用域本身的所有协程执行。

而您可以考虑针对存在时间需要比当前作用域更长的工作注入一个 `CoroutineScope`。

```kotlin
// DO inject an external scope instead of using GlobalScope.
// GlobalScope can be used indirectly. Here as a default parameter makes sense.
class ArticlesRepository(
    private val articlesDataSource: ArticlesDataSource,
    private val externalScope: CoroutineScope = GlobalScope,
    private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    // As we want to complete bookmarking the article even if the user moves
    // away from the screen, the work is done creating a new coroutine
    // from an external scope
    suspend fun bookmarkArticle(article: Article) {
        externalScope.launch(defaultDispatcher) {
            articlesDataSource.bookmarkArticle(article)
        }
            .join() // Wait for the coroutine to complete
    }
}

// DO NOT use GlobalScope directly
class ArticlesRepository(
    private val articlesDataSource: ArticlesDataSource,
) {
    // As we want to complete bookmarking the article even if the user moves away
    // from the screen, the work is done creating a new coroutine with GlobalScope
    suspend fun bookmarkArticle(article: Article) {
        GlobalScope.launch {
            articlesDataSource.bookmarkArticle(article)
        }
            .join() // Wait for the coroutine to complete
    }
}
```



## 协程设为可取消

在协程的 `Job` 被取消后，相应协程在挂起或检查是否存在取消操作之前不会被取消。如果您在协程中执行阻塞操作，请确保相应协程是可取消的。

例如，如果您要从磁盘读取多个文件，请先检查协程是否已取消，然后再开始读取每个文件。检查是否取消的一种方法是调用 [`ensureActive`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html) 函数。

`kotlinx.coroutines` 中的所有挂起函数（例如 `withContext` 和 `delay`）都是可取消的。如果协程调用这些函数，无需执行任何其他操作。

[Cancellation in coroutines. ](https://medium.com/androiddevelopers/cancellation-in-coroutines-aa6b90163629)



## 异常处理

未处理协程中抛出的异常可能会导致应用崩溃。如果可能会发生异常，请在使用 `viewModelScope` 或 `lifecycleScope` 创建的任何协程主体中捕获相应异常。

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
) : ViewModel() {

    fun login(username: String, token: String) {
        viewModelScope.launch {
            try {
                loginRepository.login(username, token)
                // Notify view user logged in successfully
            } catch (exception: IOException) {
                // Notify view login attempt failed
            }
        }
    }
}
```

