启动一个app涉及的三种通信方式



子线程通过sendMessage把消息发送到MessageQueue，主线程在activityThread.main里面会调用looper.loop开启一个死循环从MessageQueue取消息，handleMessage  



1. 线程切换的原理

   Handler: 显示

   子线程： handler.sendMessage -> queue.enqueueMessage : messageQueue

   主线程：activityThread.main -> looper.loop -> queue.next -> handler.handleMessage

2. handler内存泄漏，最终是谁持有的activity

   enqueueMessage的时候msg持有了Handler

   activityThread -》looper -》messageQueue -》msg -》 handler -》activity

   匿名内部类持有外部类对象

   msg没有被处理的时候，内存泄漏

   

3. looper什么时候进入循环

   WMS，刷新，点击，生命周期：Message

   

4. handlerThread原理（不是线程中创建的就是什么线程对应的handler）

   创建handler需要依赖looper

   

5. 子线程中维护的looper，消息队列无消息的时候的处理方案是什么？若是主线程

   调用quit释放线程

   

6. Handler如何处理发送延迟消息

   

7. 使用Message时应该如何创建它

   

8. handler没有消息处理是阻塞的还是非阻塞的？为什么不会有ANR产生？


