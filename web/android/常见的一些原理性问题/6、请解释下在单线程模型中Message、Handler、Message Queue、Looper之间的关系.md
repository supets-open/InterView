[浅谈单线程模型中Message、Handler、Message Queue、Looper之前的关系](https://blog.csdn.net/xuxian361/article/details/49232055)



简单的说，Handler获取当前线程中的looper对象，looper用来从存放Message的MessageQueue中取出Message，再有Handler进行Message的分发和处理.
Message Queue(消息队列)：

用来存放通过Handler发布的消息，通常附属于某一个创建它的线程，可以通过Looper.myQueue()得到当前线程的消息队列
Handler：

可以发布或者处理一个消息或者操作一个Runnable，通过Handler发布消息，消息将只会发送到与它关联的消息队列，然也只能处理该消息队列中的消息
Looper：

是Handler和消息队列之间通讯桥梁，程序组件首先通过Handler把消息传递给Looper，Looper把消息放入队列。Looper也把消息队列里的消息广播给所有的
Handler：Handler接受到消息后调用handleMessage进行处理
Message：

消息的类型，在Handler类中的handleMessage方法中得到单个的消息进行处理。

在单线程模型下，为了线程通信问题，Android设计了一个Message Queue(消息队列)， 线程间可以通过该Message Queue并结合Handler和Looper组件进行信息交换。
