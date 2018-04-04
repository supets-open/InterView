[Android之Activity系列总结(三)--Activity的四种启动模式 ](https://www.cnblogs.com/jycboy/p/6367829.html)

[Android之Activity系列总结(二)--任务和返回栈 ](http://www.cnblogs.com/jycboy/p/6367330.html)

[Android之Activity系列总结（一）--Activity概览 ](http://www.cnblogs.com/jycboy/p/6367282.html)



简介

通过设置ActivityManifestActivity_launchMode可以设置Activity的启动模式。

默认情况下，使用启动模式：standard。

同时，launchMode可以通过Intent flags的改变在运行时被复写，比
 如 Intent flags FLAG_ACTIVITY_SINGLE_TOP, FLAG_ACTIVITY_NEW_TASK,
FLAG_ACTIVITY_MULTIPLE_TASK.


四种模式

standard

默认模式，会在启动时创建一个新实例，创建的模式也可以随Intent.FLAG_ACTIVITY_NEW_TASK而改变
 
singleTop
 
当启动activity时，有相同的activity在前台与用户交互，那就复用这个activity，这个实例会被调用Activity.onNewIntent()。

singleTask

在启动activity时，若有一个运行着这个activity的task，那这个activity实例会被调到前台，并调用Activity.onNewIntent() ，启动实例的Intent的flag会被设置Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT . singleTask是singleTop的一个扩展集。This is a superset of the singleTop mode, where if there is already an instance of the activity being started at the top of the stack, it will receive the Intent as described there (without the FLAG_ACTIVITY_BROUGHT_TO_FRONT flag set). See the Tasks and Back Stack document for more details about tasks.
 

singleInstance

 
开辟一个只允许一个activity实例在里头运行的task. 如果用同样的intent再次启动这个activity，task会被调到前台，其Activity.onNewIntent() 会被调用. 如果这个activity实例要启动一个新activity，那么这个新activity会在一个新task中运行.
 

