
![对象状态转化图](https://raw.githubusercontent.com/supets-open/InterView/master/web/image/Activity和Fragment比较.png)


总之一句话，在创建的过程中，是Activity带领着Fragment，在销毁的过程中，是Fragment带领着Activity。

在创建的过程中，是Activity带领Fragment执行生命周期中的方法，所以生命周期方法的执行顺序是这样的：

	Activity--onCreate();
	Fragment--onAttach();
	Fragment--onCreate();
	Fragment--onCreateView();
	Fragment--onActivityCreated();
	
	Activity--onStart();
	Fragment--onStart();
	Activity--onResume();
	Fragment--onResume();

当销毁的时候，春江水暖鸭先知，当然是Fragment先感知到，于是销毁的时候就是Fragment带领Activity：

	Fragment--onPause();
	Activity--onPause();
	Fragment--onStop();
	Activity--onStop();
	Fragment--onDestroyView();
	Fragment--onDestroy();
	Fragment--onDetach();
	Activity--onDestroy();


Activity比Fragment效率高。因为Activity可以单独使用，但是Fragment要放在Activity中去使用。

Fragment是为了解决同一款安卓软件不能在分辨率和不同屏幕的载体中使用的问题而产生的软件。可以把Fragment当成Activity的一个界面的一个组成部分，甚至Activity的界面可以完全有不同的Fragment组成，更帅气的是Fragment拥有自己的生命周期和接收、处理用户的事件，这样就不必在Activity写一堆控件的事件处理的代码了。更为重要的是，你可以动态的添加、替换和移除某个Fragment。





