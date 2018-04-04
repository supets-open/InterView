一般情况下比如说有两个activity,分别叫A,B。
当在A 里面激活B 组件的时候, A会调用onPause()方法,然后B调用onCreate() ,onStart(), onResume()。
这个时候B覆盖了A的窗体, A会调用onStop()方法。
如果B是个透明的窗口,或者是对话框的样式, 就不会调用A的onStop()方法。