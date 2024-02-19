# 广播简介

Android中的每个应用程序都可以对自己感兴趣的广播进行注册，这样该程序就只会收到自己所关心的广播内容，这些广播可能是来自于系统的，也可能是来自于其他应用程序的。Android提供了一套完整的API，允许应用程序自由地发送和接收广播。

## 标准广播

标准广播（normal broadcasts）是一种完全异步执行的广播，在广播发出之后，所有的BroadcastReceiver几乎会在同一时刻收到这条广播消息，因此它们之间没有任何先后顺序可言。这种广播的效率会比较高，但同时也意味着它是无法被截断的。

## 有序广播

有序广播（ordered broadcasts）则是一种同步执行的广播，在广播发出之后，同一时刻只会有一个BroadcastReceiver能够收到这条广播消息，当这个BroadcastReceiver中的逻辑执行完毕后，广播才会继续传递。所以此时的BroadcastReceiver是有先后顺序的，优先级高的BroadcastReceiver就可以先收到广播消息，并且前面的BroadcastReceiver还可以截断正在传递的广播，这样后面的BroadcastReceiver就无法收到广播消息了。

# 接收系统广播

可以根据自己感兴趣的广播，自由地注册BroadcastReceiver，这样当有相应的广播发出时，相应的BroadcastReceiver就能够收到该广播，并可以在内部进行逻辑处理。

注册BroadcastReceiver的方式一般有两种：在代码中注册和在AndroidManifest.xml中注册。其中前者也被称为动态注册，后者也被称为静态注册。

创建一个BroadcastReceiver，只需新建一个类，让它继承自BroadcastReceiver，并重写父类的onReceive()方法就行了。这样当有广播到来时，onReceive()方法就会得到执行，具体的逻辑就可以在这个方法中处理。

## 动态注册

通过动态注册的方式编写一个能够监听时间变化的程序

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var timeChangeReceiver: TimeChangeRecevier
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState) 
        setContentView(R.layout.activity_main)         
        val intentFilter = IntentFilter()         
        intentFilter.addAction("android.intent.action.TIME_TICK")        
        timeChangeReceiver = TimeChangeReceiver()         
        registerReceiver(timeChangeReceiver, intentFilter) 
    }
    
    override fun onDestroy() {         
    super.onDestroy()         
    unregisterReceiver(timeChangeReceiver)     
}     

	inner class TimeChangeReceiver : BroadcastReceiver() {         
    	override fun onReceive(context: Context, intent: Intent) {            
        	Toast.makeText(context, "Time has changed", Toast.LENGTH_SHORT).show()        
    	}     
	} 
}
```

在MainActivity中定义了一个内部类TimeChangeReceiver，这个类是继承BroadcastReceiver的，并重写了父类的onReceive()方法。这样每当系统时间发生变化时，onReceive()方法就会得到执行，这里只是简单地使用Toast提示了一段文本信息。然后观察onCreate()方法，首先我们创建了一个IntentFilter的实例，并给它添加了一个值android.intent.action.TIME_TICK的action，为什么要添加这个值呢？因为当系统时间发生变化时，系统发出的正是一条值为android.intent.action.TIME_TICK的广播，也就是说我们的BroadcastReceiver想要监听什么广播，就在这里添加相应的action。接下来创建了一个TimeChangeReceiver的实例，然后调用registerReceiver()方法进行注册，将TimeChangeReceiver的实例和IntentFilter的实例都传了进去，这样TimeChangeReceiver就会收到所有值为android.intent.action.TIME_TICK的广播，也就实现了监听系统时间变化的功能。

动态注册的BroadcastReceiver一定要取消注册才行，这里我们是在onDestroy()方法中通过调用unregisterReceiver()方法来实现的。

## 静态注册

动态注册的BroadcastReceiver可以自由地控制注册与注销，在灵活性方面有很大的优势。但是它存在着一个缺点，即必须在程序启动之后才能接收广播，因为注册的逻辑是写在onCreate()方法中的。那么有没有什么办法可以让程序在未启动的情况下也能接收广播呢？这就需要使用静态注册的方式了。

# 发送自定义广播

## 发送标准广播

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {     
    override fun onReceive(context: Context, intent: Intent) {         
        Toast.makeText(context, "received in MyBroadcastReceiver",            Toast.LENGTH_SHORT).show()     
    } 
}

button.setOnClickListener {             
    val intent = Intent("com.example.broadcasttest.MY_BROADCAST")           
    intent.setPackage(packageName)             
    sendBroadcast(intent)         
}
```

首先构建了一个Intent对象，并把要发送的广播的值传入。然后调用Intent的setPackage()方法，并传入当前应用程序的包名。packageName是getPackageName()的语法糖写法，用于获取当前应用程序的包名。最后调用sendBroadcast()方法将广播发送出去，这样所有监听com.example.broadcasttest.MY_BROADCAST这条广播的BroadcastReceiver就会收到消息了。

在Android8.0系统之后，静态注册的BroadcastReceiver是无法接收隐式广播的，而默认情况下我们发出的自定义广播恰恰都是隐式广播。因此这里一定要调用setPackage()方法，指定这条广播是发送给哪个应用程序的，从而让它变成一条显式广播，否则静态注册的BroadcastReceiver将无法接收到这条广播。还可以在Intent中携带一些数据传递给相应的BroadcastReceiver。

## 发送有序广播

发送有序广播只需要改动一行代码，即将sendBroadcast()方法改成sendOrderedBroadcast()方法。sendOrderedBroadcast()方法接收两个参数：第一个参数仍然是Intent；第二个参数是一个与权限相关的字符串，这里传入null就行了。

如何设定BroadcastReceiver的先后顺序呢？当然是在注册的时候进行设定。通过android:priority属性给BroadcastReceiver设置了优先级，优先级比较高的BroadcastReceiver就可以先收到广播。这里将MyBroadcastReceiver的优先级设成了100，以保证它一定会在AnotherBroadcastReceiver之前收到广播。

如果在onReceive()方法中调用了abortBroadcast()方法，就表示将这条广播截断，后面的BroadcastReceiver将无法再接收到这条广播。

# 广播实践：强制下线

强制用户下线的逻辑并不是写在MainActivity里的，而是应该写在接收这条广播的BroadcastReceiver里。这样强制下线的功能就不会依附于任何界面了，不管是在程序的任何地方，只要发出这样一条广播，就可以完成强制下线的操作了。

需要创建一个BroadcastReceiver来接收这条强制下线广播。唯一的问题就是，应该在哪里创建呢？由于BroadcastReceiver中需要弹出一个对话框来阻塞用户的正常操作，但如果创建的是一个静态注册的BroadcastReceiver，是没有办法在onReceive()方法里弹出对话框这样的UI控件的，而我们显然也不可能在每个Activity中都注册一个动态的BroadcastReceiver。那么到底应该怎么办呢？答案其实很明显，只需要在BaseActivity中动态注册一个BroadcastReceiver就可以了，因为所有的Activity都继承自BaseActivity。

# Kotlin

## 高阶函数

## 内联函数