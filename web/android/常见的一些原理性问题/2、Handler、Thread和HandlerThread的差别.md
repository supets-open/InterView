

# 重要
> Handler可以在线程中声明，实际上Handler依赖于当前线程中messageQueue和Looper（每个线程可以且只能有一个），也就是说你在非UI线程中构造的Handler，它的循环和事件处理仍然是在当前线程中执行的，不能更新UI 


[Thread、Handler和HandlerThread关系何在？](https://blog.csdn.net/ly502541243/article/details/52414637)

[HandlerThread和Handler之间的关系](https://blog.csdn.net/solarsaber/article/details/45969435)

[Android中使用Handler机制更新UI的三种解决方案 ](https://blog.csdn.net/CHENYUFENG1991/article/details/46910675)

[Handler消息传递机制（子线程中传递new Handler和主线程中new Handle传递消息） ](https://blog.csdn.net/ShareUs/article/details/50787890)

[Handler一定要在主线程实例化吗?new Handler()和new Handler(Looper.getMainLooper())的区别](https://blog.csdn.net/thanklife/article/details/17006865)