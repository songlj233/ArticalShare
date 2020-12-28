# 学习笔记

## 对handler的理解

### 基本介绍

Handler消息机制是Android中用于线程间通讯的一种解决方案，简单说就是在主线程中统一调度各个线程发送的消息。

在应用启动的时候，有一个叫做ActivityThread的类，这个类其实就是这个应用的main入口，他在这个main函数里面创建了这个Handler，一起创建的还有Looper，用于消息事件的驱动（Android应用程序启动后没有关闭，也是因为这个Looper创建了一个for(;;)的死循环）。

### 工作流程

介绍工作流程前，要先讲一下配合Handler一起使用的MessageQueen，这是一个消息队列，他是一个按时间排序的优先级队列，他的本质其实就是单向链表。这个消息队列接收Handler发送过来的消息，根据时间节点来确定插入MessageQueen的位置，然后由Looper.loop()取出队头的消息(消息要满足：消息的执行时间≥系统的当前时间),并进行分发调度。

一个线程有且只有一个Looper，在应用启动的时候，通过prepare()函数创建的，保存在ThreadLocal中(ThreadLocal里面有一个ThreadLocalMap，跟HashMap类似的保存键值对)，如果Looper已经存在则不会重复创建，保证了一个线程只能有一个Looper。

但是一个线程可以有多个Handler，但是都对应同一个消息队列，由同一个Looper统一调度。

## 同步屏障机制（选讲）

消息有分同步和异步。当消息需要优先执行时（点击事件，布局绘制），可以发送同步屏障postSyncBarrier，就可以优先处理异步消息了。

举个例子，当有个优先级比较高的消息的时候，比如点击按钮或者个更新布局，这时候会发送一个同步屏障，然后再给Handler发送消息（消息的target为null）。当消息队列用next获取消息的时候，如果消息的target为null，就会遍历消息队列中的异步消息优先执行。如果不需要同步屏障的时候，还需要将它移除。

## 总结

1. AsyncTask他的本质也是使用了Handler的消息机制
2. 使用Handler需要避免内存泄露
3. looper死循环为什么不会导致anr