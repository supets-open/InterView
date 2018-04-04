[Activity 知识梳理(3) - Activity状态保存和恢复](https://www.jianshu.com/p/715333d87738)

[Android学习总结——Activity状态保存和恢复](https://www.cnblogs.com/xch-yang/p/5762722.html)

[保存/恢复Activity和Fragment状态的最佳实践](https://zhuanlan.zhihu.com/p/22141193?utm_source=tuicool&utm_medium=referral)


当Activity的onSaveInstanceState被调用的时候，Activity将会从View 层次(View Hierachy)中的每一个View中自动搜集View的状态。请注意，只会搜集实现了View状态保存/恢复的内部方法的View的数据。一旦onRestoreInstanceState被调用,Activity将会将这些搜集到的数据一对一的返还给View 层次里在搜集的时候提供了同样的android:id属性的View。

这就是为什么尽管Activity已经被销毁，而我们并没有做一些特别的事情来保存状态，但是EditText中键入的文本仍然能够呈现的原因。这并不是什么魔法，这些View 的状态已经被自动的保存和恢复回来了。

这也是为什么View 在没有被设置android:id属性的时候不能保存和恢复自己的状态的原因。

尽管这些View 的状态被自动的保存了，但是Activity的成员变量并不会有同样的效果。这些成员变量会被和Activity一起销毁。你可以手动的保存和恢复它们，通过onSaveInstanceState和onRestoreInstanceState方法。


* 理解在Fragment的状态被保存/恢复的时候发生了什么

你必须分别地通过onSaveInstanceState和onActivityCreated方法，手动的保存和恢复这些变量。 请注意在Fragment里面没有onRestoreInstanceState方法存在。

对于Fragment来说，有一些特殊情况不同于Activity,我觉得你需要知道这些情况。一旦Fragment从后退栈中返回，它的View 会被销毁，并重新创建

![对象状态转化图](https://raw.githubusercontent.com/supets-open/InterView/master/web/image/fragment销毁.jpg)

这这种情况下，Fragment并不会被销毁，只有Fragment中的View 会被销毁。结果是，并不会发生任何实例状态的保存。那上面展示在Fragment生命周期重新创建View的时候发生了什么？

别惊讶，因为Android是这样设计的。在这种情况下，Fragment中的View 状态的保存/恢复会被内部调用。结果就是，每一个实现了内部View 状态保存/恢复的View ，将会被自动的保存并且恢复状态，例如带有android:freezesText="true"属性的EditText或者TextView。就像之前完美显示的一样。


所以Fragment状态保存/恢复最佳实践的第一条件就是...
应用中使用的每一个单独的View都必须在内部实现状态的保存和恢复

Android内部通过onSaveInstanceState 和 onRestoreInstanceState 方法提供了保存和恢复View 状态的机制。开发者的任务就是实现它。
