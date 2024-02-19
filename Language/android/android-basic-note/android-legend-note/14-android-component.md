### Android四大组件如何协同工作

Activity作为人机交互的第一界面，负责向用户展示信息和处理结果，而这些信息的来源，可以是通过资源获取，也可以通过Content Provider来获取其他应用的信息，或是Service从后台计算、下载、处理的结果，当然也可以是通过BroadCast Reciever获取到的广播信息。同时，Android系统还提供了一个信使——Intent，作为信息传递的载体。组件与组件之间通过Intent来通信、传递信息、交换数据。

### 应用运行上下文对象

Android系统的上下文对象，即在Context中，为我们封装了这样一个“语境”。Activity、Service、Application都是继承自Context。

Android应用程序会在如下所示的几个时间点创建应用上下文Context。

- 创建Application

- 创建Activity

- 创建Service

  

创建Context的时机就是在创建Context的实现类的时候。当应用程序第一次启动时，Android系统都会创建一个Application对象，同时创建Application Context，所有的组件都共同拥有这样一个Context对象，这个应用上下文对象贯穿整个应用进程的生命周期，为应用全局提供了功能和环境支持。而创建Activity和Service组件时，系统也会给它们提供运行的上下文环境，即创建Activity实例、Service实例的Context对象。在Activity中获取Context对象时，可以直接使用this，而在匿名内部类中，就必须指定XXXXActivity.this才可以获得该Activity的Context对象。当然，你也可以通过getApplicationContext()方法来获取整个App的Context，但是通过getApplicationContext()方法获得的是整个应用的上下文引用，这与某个组件的上下文引用，在某些时候还是有区别的。