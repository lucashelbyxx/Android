# WebView的用法

在应用程序里展示一些网页。不允许打开系统浏览器。WebView控件，借助它我们就可以在自己的应用程序里嵌入一个浏览器，从而非常轻松地展示各种各样的网页。

```kotlin
class MainActivity : AppCompatActivity() {     
    override fun onCreate(savedInstanceState: Bundle?) {         
        super.onCreate(savedInstanceState)         
        setContentView(R.layout.activity_main)         
        webView.settings.javaScriptEnabled=true         
        webView.webViewClient = WebViewClient()         
        webView.loadUrl("https://www.baidu.com")     
    } 
}
```

调用了setJavaScriptEnabled()方法，让WebView支持JavaScript脚本。调用了WebView的setWebViewClient()方法，并传入了一个WebViewClient的实例。这段代码的作用是，当需要从一个网页跳转到另一个网页时，我们希望目标网页仍然在当前WebView中显示，而不是打开系统浏览器。

# 使用HTTP访问网络

WebView已经在后台帮我们处理好了发送HTTP请求、接收服务器响应、解析返回数据，以及最终的页面展示这几步工作，只不过它封装得实在是太好了，反而使得我们不能那么直观地看出HTTP到底是如何工作的。因此，接下来就让我们通过手动发送HTTP请求的方式更加深入地理解这个过程。

## 使用HttpURLConnection

```kotlin
// 取HttpURLConnection的实例，一般只需创建一个URL对象，并传入目标的网络地址，然后调用一下openConnection()方法即可
val url = URL("https://www.baidu.com") 
val connection = url.openConnection() as HttpURLConnectio

// 得到了HttpURLConnection的实例之后，我们可以设置一下HTTP请求所使用的方法。常用的方法主要有两个：GET和POST。GET表示希望从服务器那里获取数据，而POST则表示希望提交数据给服务器
connection.requestMethod = "GET"

// 设置连接超时、读取超时的毫秒数
connection.connectTimeout = 8000 
connection.readTimeout = 8000

// 调用getInputStream()方法就可以获取到服务器返回的输入流
val input = connection.inputStream 

// 下面对获取到的输入流进行读取                
val reader = BufferedReader(InputStreamReader(input))                reader.use {                     
    reader.forEachLine {                         
        response.append(it)                     
    }                 
}

// 调用disconnect()方法将这个HTTP连接关闭
connection.disconnect()

// 提交数据给服务器，只需要将HTTP请求的方法改成POST，并在获取输入流之前把要提交的数据写出即可。每条数据都要以键值对的形式存在，数据与数据之间用“&”符号隔开。
connection.requestMethod = "POST" 
val output = DataOutputStream(connection.outputStream) 
output.writeBytes("username=admin&password=123456")
```



## 使用OkHttp

有许多出色的网络通信库都可以替代原生的HttpURLConnection，而其中OkHttp无疑是做得最出色的一个。在使用OkHttp之前，我们需要先在项目中添加OkHttp库的依赖。编辑app/build.gradle文件，在dependencies闭包中添加如下内容：

```
dependencies {     
    ...     
    implementation 'com.squareup.okhttp3:okhttp:4.1.0' 
}
```

添加上述依赖会自动下载两个库：一个是OkHttp库，一个是Okio库，后者是前者的通信基础。

```kotlin
// OkHttp具体用法
// 创建一个OkHttpClient实例
val client = OkHttpClient()

// 想要发起一条HTTP请求，需要创建一个Request对象
val request = Request.Builder().url("https://www.baidu.com").build()

// 调用OkHttpClient的newCall()方法来创建一个Call对象，并调用它的execute()方法来发送请求并获取服务器返回的数据
val response = client.newCall(request).execute()

// Response对象就是服务器返回的数据了，我们可以使用如下写法来得到返回的具体内容
val responseData = response.body?.string() 


// 如果是发起一条POST请求，会比GET请求稍微复杂一点，我们需要先构建一个Request Body对象来存放待提交的参数
val requestBody = FormBody.Builder()         
        .add("username", "admin")         
        .add("password", "123456")         
        .build()

// 在Request.Builder中调用一下post()方法，并将RequestBody对象传入
val request = Request.Builder()         
        .url("https://www.baidu.com")         
        .post(requestBody)         
        .build()
```

在这个方法中同样还是先开启了一个子线程，然后在子线程里使用OkHttp发出一条HTTP请求，请求的目标地址还是百度的首页，OkHttp的用法也正如前面所介绍的一样。最后仍然调用了showResponse()方法，将服务器返回的数据显示到界面上。

# 解析XML格式数据

在网络上传输数据时最常用的格式有两种：XML和JSON。

## Pull解析方式

```kotlin
private fun parseXMLWithPull(xmlData: String) {         
    try {             
        val factory = XmlPullParserFactory.newInstance()             
        val xmlPullParser = factory.newPullParser()             
        xmlPullParser.setInput(StringReader(xmlData))             
        var eventType = xmlPullParser.eventType             
        var id = ""             
        var name = ""             
        var version = ""             
        while (eventType != XmlPullParser.END_DOCUMENT) {                 
            val nodeName = xmlPullParser.name                 
            when (eventType) {                     
                // 开始解析某个节点                    
                XmlPullParser.START_TAG -> {                         
                    when (nodeName) {                             
                        "id" -> id = xmlPullParser.nextText()                            
                        "name" -> name = xmlPullParser.nextText()                            
                        "version" -> version = xmlPullParser.nextText()                        
                    }                     
                }                     
                // 完成解析某个节点                    
                XmlPullParser.END_TAG -> {                         
                    if ("app" == nodeName) {                             
                        Log.d("MainActivity", "id is $id")                   
                        Log.d("MainActivity", "name is $name")                            
                        Log.d("MainActivity", "version is $version")                        
                    }                     
                }                 
            }                 
            eventType = xmlPullParser.next()             
        }         
    } catch (e: Exception) {             
        e.printStackTrace()         
    }     
} 
```

这里首先要创建一个XmlPullParserFactory的实例，并借助这个实例得到XmlPullParser对象，然后调用XmlPullParser的setInput()方法将服务器返回的XML数据设置进去，之后就可以开始解析了。解析的过程也非常简单，通过getEventType()可以得到当前的解析事件，然后在一个while循环中不断地进行解析，如果当前的解析事件不等于XmlPullParser.END_DOCUMENT，说明解析工作还没完成，调用next()方法后可以获取下一个解析事件。在while循环中，我们通过getName()方法得到了当前节点的名字。如果发现节点名等于id、name或version，就调用nextText()方法来获取节点内具体的内容，每当解析完一个app节点，就将获取到的内容打印出来。

## SAX解析方式

# 解析JSON格式数据

比起XML，JSON的主要优势在于它的体积更小，在网络上传输的时候更省流量。但缺点在于，它的语义性较差，看起来不如XML直观。

## 使用JSONObject

```kotlin
private fun parseJSONWithJSONObject(jsonData: String) {         
    try {             
        val jsonArray = JSONArray(jsonData)             
        for (i in 0 until jsonArray.length()) {                 
            val jsonObject = jsonArray.getJSONObject(i)                 
            val id = jsonObject.getString("id")                 
            val name = jsonObject.getString("name")                 
            val version = jsonObject.getString("version")                 
            Log.d("MainActivity", "id is $id")                 
            Log.d("MainActivity", "name is $name")                 
            Log.d("MainActivity", "version is $version")             
        }         
    } catch (e: Exception) {             
        e.printStackTrace()        
    }     
}
```

解析JSON的代码真的非常简单，由于我们在服务器中定义的是一个JSON数组，因此这里首先将服务器返回的数据传入一个JSONArray对象中。然后循环遍历这个JSONArray，从中取出的每一个元素都是一个JSONObject对象，每个JSONObject对象中又会包含id、name和version这些数据。接下来只需要调用getString()方法将这些数据取出，并打印出来即可。

## 使用GSON

# 网络请求回调的实现方式

因为一个应用程序很可能会在许多地方都使用到网络功能，而发送HTTP请求的代码基本是相同的，如果我们每次都去编写一遍发送HTTP请求的代码，这显然是非常差劲的做法。通常情况下我们应该将这些通用的网络操作提取到一个公共的类里，并提供一个通用方法，当想要发起网络请求的时候，只需简单地调用一下这个方法即可。

网络请求通常属于耗时操作，而sendHttpRequest()方法的内部并没有开启线程，这样就有可能导致在调用sendHttpRequest()方法的时候主线程被阻塞。如果我们在sendHttpRequest()方法中开启一个线程来发起HTTP请求，服务器响应的数据是无法进行返回的。这是由于所有的耗时逻辑都是在子线程里进行的，sendHttpRequest()方法会在服务器还没来得及响应的时候就执行结束了，当然也就无法返回响应的数据了。遇到这种情况时应该怎么办呢？其实解决方法并不难，只需要使用编程语言的回调机制就可以了。

```kotlin
object HttpUtil {     
    fun sendHttpRequest(address: String, listener: HttpCallbackListener) {   
        thread {           
            var connection: HttpURLConnection? = null             
            try {                 
                val response = StringBuilder()                 
                val url = URL(address)                 
                connection = url.openConnection() as HttpURLConnection       
                connection.connectTimeout = 8000                 
                connection.readTimeout = 8000                 
                val input = connection.inputStream                 
                val reader = BufferedReader(InputStreamReader(input))                reader.use {                     
                    reader.forEachLine {                         
                        response.append(it)                     
                    }                 
                }                 
                // 回调onFinish()方法                
                listener.onFinish(response.toString())             
            } catch (e: Exception) {                 
                e.printStackTrace()                 
                // 回调onError()方法                
                listener.onError(e)             
            } finally {                 
                connection?.disconnect()             
            }         
        }     
    } 
}

// 以后每当需要发起一条HTTP请求的时候，就可以这样写
val address = "https://www.baidu.com" 
val response = HttpUtil.sendHttpRequest(address)

// 首先需要定义一个接口，比如将它命名成HttpCallbackListener
interface HttpCallbackListener {     
    fun onFinish(response: String)     
    fun onError(e: Exception) 
}

HttpUtil.sendHttpRequest(address, object : HttpCallbackListener {    
    override fun onFinish(response: String) {         
        // 得到服务器返回的具体内容    
    }     
    override fun onError(e: Exception) {         
        // 在这里对异常情况进行处理    
    } 
})
```

先给sendHttpRequest()方法添加了一个HttpCallbackListener参数，并在方法的内部开启了一个子线程，然后在子线程里执行具体的网络操作。注意，子线程中是无法通过return语句返回数据的，因此这里我们将服务器响应的数据传入了HttpCallbackListener的onFinish()方法中，如果出现了异常，就将异常原因传入onError()方法中。

上述使用HttpURLConnection的写法总体来说还是比较复杂的，使用OkHttp会变得简单。

```kotlin
object HttpUtil {     
    ...     
    fun sendOkHttpRequest(address: String, callback: okhttp3.Callback) {     
        val client = OkHttpClient()         
        val request = Request.Builder()             
            .url(address)             
            .build()         
        client.newCall(request).enqueue(callback)     
    } 
} 

// 用sendOkHttpRequest()方法的时候就可以这样写
HttpUtil.sendOkHttpRequest(address, object : Callback {     
    override fun onResponse(call: Call, response: Response) {         
        // 得到服务器返回的具体内容        
        val responseData = response.body?.string()     
    }     
    override fun onFailure(call: Call, e: IOException) {         
        // 在这里对异常情况进行处理    
    } 
}
)
```

sendOkHttpRequest()方法中有一个okhttp3.Callback参数，这个是OkHttp库中自带的回调接口，类似于我们刚才自己编写的HttpCallbackListener。然后在client.newCall()之后没有像之前那样一直调用execute()方法，而是调用了一个enqueue()方法，并把okhttp3.Callback参数传入。相信聪明的你已经猜到了，OkHttp在enqueue()方法的内部已经帮我们开好子线程了，然后会在子线程中执行HTTP请求，并将最终的请求结果回调到okhttp3.Callback当中。

不管是使用HttpURLConnection还是OkHttp，最终的回调接口都还是在子线程中运行的，因此我们不可以在这里执行任何的UI操作，除非借助runOnUiThread()方法来进行线程转换。

# 最好用的网络库：Retrofit

OkHttp侧重的是底层通信的实现，而Retrofit侧重的是上层接口的封装。事实上，Retrofit就是Square公司在OkHttp的基础上进一步开发出来的应用层网络通信库，使得我们可以用更加面向对象的思维进行网络操作。

# Kotlin

什么是协程呢？它其实和线程是有点类似的，可以简单地将它理解成一种轻量级的线程。要知道，我们之前所学习的线程是非常重量级的，它需要依靠操作系统的调度才能实现不同线程之间的切换。而使用协程却可以仅在编程语言的层面就能实现不同协程之间的切换，从而大大提升了并发编程的运行效率。

```kotlin
fun foo() {     
    a()     
    b()     
    c() 
} 
fun bar() {     
    x()     
    y()     
    z() 
}
```

在没有开启线程的情况下，先后调用foo()和bar()这两个方法，那么理论上结果一定是a()、b()、c()执行完了以后，x()、y()、z()才能够得到执行。而如果使用了协程，在协程A中去调用foo()方法，协程B中去调用bar()方法，虽然它们仍然会运行在同一个线程当中，但是在执行foo()方法时随时都有可能被挂起转而去执行bar()方法，执行bar()方法时也随时都有可能被挂起转而继续执行foo()方法，最终的输出结果也就变得不确定了。

协程允许我们在单线程模式下模拟多线程编程的效果，代码执行时的挂起与恢复完全是由编程语言来控制的，和操作系统无关。这种特性使得高并发程序的运行效率得到了极大的提升。

## 协程的基本用法

Kotlin并没有将协程纳入标准库的API当中，而是以依赖库的形式提供的。所以如果我们想要使用协程功能，需要先在app/build.gradle文件当中添加如下依赖库：

```
dependencies {   
   ...   
   implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1"  	implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1"
}
```

## 更多的作用域构建器

## 使用协程简化回调的写法