[显式和隐式启动Activity、Service](https://blog.csdn.net/what93/article/details/50469255)


[简要解释一下activity、 intent 、intent filter、service、Broadcase、BroadcaseReceiver](https://blog.csdn.net/hanchendong/article/details/51420129)


Activity 和 Service 都是 Android 四大组件之一。他俩都是
Context 类的子类 ContextWrapper 的子类，因此他俩可以算是兄弟关系吧。不过兄弟俩各有各自的本领，Activity
负责用户界面的显示和交互，Service 负责后台任务的处理。Activity 和 Service 之间可以通过 Intent 传递数据，因此
可以把 Intent 看作是通信使者