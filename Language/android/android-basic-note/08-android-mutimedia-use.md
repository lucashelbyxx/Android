# 使用通知

## 创建通知渠道

Android 8.0系统引入了通知渠道这个概念。每条通知都要属于一个对应的渠道。每个应用程序都可以自由地创建当前应用拥有哪些通知渠道，但是这些通知渠道的控制权是掌握在用户手上的。用户可以自由地选择这些通知渠道的重要程度，是否响铃、是否振动或者是否要关闭这个渠道的通知。

创建通知渠道的详细步骤。首先需要一个NotificationManager对通知进行管理，可以通过调用Context的getSystemService()方法获取。getSystemService()方法接收一个字符串参数用于确定获取系统的哪个服务，这里我们传入Context.NOTIFICATION_SERVICE即可。接下来要使用NotificationChannel类构建一个通知渠道，并调用NotificationManager的createNotificationChannel()方法完成创建。由于NotificationChannel类和createNotificationChannel()方法都是Android 8.0系统中新增的API，因此我们在使用的时候还需要进行版本判断才可以。创建一个通知渠道至少需要渠道ID、渠道名称以及重要等级这3个参数，其中渠道ID可以随便定义，只要保证全局唯一性就可以。渠道名称是给用户看的，需要可以清楚地表达这个渠道的用途。通知的重要等级主要有IMPORTANCE_HIGH、IMPORTANCE_DEFAULT、IMPORTANCE_LOW、IMPORTANCE_MIN这几种，对应的重要程度依次从高到低。不同的重要等级会决定通知的不同行为。

```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {     val channel = NotificationChannel(channelId, channelName, importance)    manager.createNotificationChannel(channel) } 
```

## 通知的基本用法

通知的用法还是比较灵活的，既可以在Activity里创建，也可以在BroadcastReceiver里创建，当然还可以在后面我们即将学习的Service里创建。相比于BroadcastReceiver和Service，在Activity里创建通知的场景还是比较少的，因为一般只有当程序进入后台的时候才需要使用通知。

创建通知的详细步骤。

首先需要使用一个Builder构造器来创建Notification对象，但问题在于，Android系统的每一个版本都会对通知功能进行或多或少的修改，API不稳定的问题在通知上凸显得尤其严重，比方说刚刚介绍的通知渠道功能在Android 8.0系统之前就是没有的。那么该如何解决这个问题呢？其实解决方案我们之前已经见过好几回了，就是使用AndroidX库中提供的兼容API。AndroidX库中提供了一个NotificationCompat类，使用这个类的构造器创建Notification对象，就可以保证我们的程序在所有Android系统版本上都能正常工作了。

```kotlin
// 先是使用Intent表达出我们想要启动NotificationActivity的“意图”，然后将构建好的Intent对象传入PendingIntent的getActivity()方法里，以得到PendingIntent的实例，接着在NotificationCompat.Builder中调用setContentIntent()方法，把它作为参数传入即可
val intent = Intent(this, NotificationActivity::class.java)            
val pi = PendingIntent.getActivity(this, 0, intent, 0
val notification = NotificationCompat.Builder(context, channelId)
.setContentTitle("This is content title")
.setContentText("This is content text")       .setSmallIcon(R.drawable.small_icon)       .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.large_icon)                                                                          .setContentIntent(pi)
.build()
  
// 调用NotificationManager的notify()方法就可以让通知显示出来了。notify()方法接收两个参数：第一个参数是id，要保证为每个通知指定的id都是不同的；第二个参数则是Notification对象                
manager.notify(1, notification)
```

实现通知的点击效果，我们还需要在代码中进行相应的设置，这就涉及了一个新的概念——PendingIntent。PendingIntent从名字上看起来就和Intent有些类似，它们确实存在不少共同点。比如它们都可以指明某一个“意图”，都可以用于启动Activity、启动Service以及发送广播等。不同的是，Intent倾向于立即执行某个动作，而PendingIntent倾向于在某个合适的时机执行某个动作。所以，也可以把PendingIntent简单地理解为延迟执行的Intent。

PendingIntent的用法同样很简单，它主要提供了几个静态方法用于获取PendingIntent的实例，可以根据需求来选择是使用getActivity()方法、getBroadcast()方法，还是getService()方法。这几个方法所接收的参数都是相同的：第一个参数依旧是Context，不用多做解释；第二个参数一般用不到，传入0即可；第三个参数是一个Intent对象，我们可以通过这个对象构建出PendingIntent的“意图”；第四个参数用于确定PendingIntent的行为，有FLAG_ONE_SHOT、FLAG_NO_CREATE、FLAG_CANCEL_CURRENT和FLAG_UPDATE_CURRENT这4种值可选，每种值的具体含义你可以查看文档，通常情况下这个参数传入0就可以了。

NotificationCompat.Builder。这个构造器还可以连缀一个setContentIntent()方法，接收的参数正是一个PendingIntent对象。因此，这里就可以通过PendingIntent构建一个延迟执行的“意图”，当用户点击这条通知时就会执行相应的逻辑。



如果我们没有在代码中对该通知进行取消，它就会一直显示在系统的状态栏上。解决的方法有两种：一种是在NotificationCompat.Builder中再连缀一个setAutoCancel()方法，一种是显式地调用NotificationManager的cancel()方法将它取消。

```kotlin
val notification = NotificationCompat.Builder(this, "normal")         
    ...         
    .setAutoCancel(true)         
    .build()

class NotificationActivity : AppCompatActivity() {     
    override fun onCreate(savedInstanceState: Bundle?) {         
        super.onCreate(savedInstanceState)         
        setContentView(R.layout.activity_notification)         
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as            NotificationManager      
        // 如果你想取消哪条通知，在cancel()方法中传入该通知的id就行了
        manager.cancel(1)     
    } 
}
```

## 通知的进阶技巧

需要在通知当中显示一段长文字，Android也是支持的，通过setStyle()方法就可以做到。使用了setStyle()方法替代setContentText()方法。在setStyle()方法中，我们创建了一个NotificationCompat.BigTextStyle对象，这个对象就是用于封装长文字信息的，只要调用它的bigText()方法并将文字内容传入就可以了。

除了显示长文字之外，通知里还可以显示一张大图片，仍然是调用的setStyle()方法，这次我们在参数中创建了一个NotificationCompat.BigPictureStyle对象，这个对象就是用于设置大图片的，然后调用它的bigPicture()方法并将图片传入。这里我事先准备好了一张图片，通过BitmapFactory的decodeResource()方法将图片解析成Bitmap对象，再传入bigPicture()方法中就可以了。

开发者只能在创建通知渠道的时候为它指定初始的重要等级，如果用户不认可这个重要等级的话，可以随时进行修改，开发者对此无权再进行调整和变更，因为通知渠道一旦创建就不能再通过代码修改了。

# 调用摄像头和相册