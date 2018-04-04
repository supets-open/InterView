## 弹出Dialog，生命值周期回调方法

弹出对话框dialog不会回调onPause，主题对话框的Activity会回调onPause


## Android应用前后台切换的判断：

1 实现一个BaseActivity，然后让其他所有Activity都继承自它，然后在生命周期函数中做相应的检测。具体检测方法如下：在Activity的onStart和onStop方法中进行计数，计数变量为count，在onStart中将变量加1，onStop中减1。


2 Android在API 14之后，在Application类中，提供了一个应用生命周期回调的注册方法，用来对应用的生命周期进行集中管理，这个接口叫registerActivityLifecycleCallbacks，可以通过它注册自己的ActivityLifeCycleCallback，每一个Activity的生命周期都会回调到这里的对应方法。其实这个注册方法的本质和我们实现BaseActivity是一样的，只是将生命周期的管理移到了Activity本身的实现中。


	public class MyApplication extends Application{  
	    public int count = 0;  
	    @Override  
	    public void onCreate() {  
	        super.onCreate();  
	  
	        registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {  
	  
	            @Override  
	            public void onActivityStopped(Activity activity) {  
	                Log.v("viclee", activity + "onActivityStopped");  
	                count--;  
	                if (count == 0) {  
	                    Log.v("viclee", ">>>>>>>>>>>>>>>>>>>切到后台  lifecycle");  
	                }  
	            }  
	  
	            @Override  
	            public void onActivityStarted(Activity activity) {  
	                Log.v("viclee", activity + "onActivityStarted");  
	                if (count == 0) {  
	                    Log.v("viclee", ">>>>>>>>>>>>>>>>>>>切到前台  lifecycle");  
	                }  
	                count++;  
	            }  
	  
	            @Override  
	            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {  
	                Log.v("viclee", activity + "onActivitySaveInstanceState");  
	            }  
	  
	            @Override  
	            public void onActivityResumed(Activity activity) {  
	                Log.v("viclee", activity + "onActivityResumed");  
	            }  
	  
	            @Override  
	            public void onActivityPaused(Activity activity) {  
	                Log.v("viclee", activity + "onActivityPaused");  
	            }  
	  
	            @Override  
	            public void onActivityDestroyed(Activity activity) {  
	                Log.v("viclee", activity + "onActivityDestroyed");  
	            }  
	  
	            @Override  
	            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {  
	                Log.v("viclee", activity + "onActivityCreated");  
	            }  
	        });  
	    }  
	}  

3 当应用切到后台的时候，运行在前台的进程由我们的app变成了桌面app，依据这一点，我们可以实现检测应用前后台切换的功能。在Activity的onStop生命周期中执行检测代码，如果发现当前运行在前台的进程不是我们自己的进程，说明应用切到了后台。


      想想为什么要在onStop中检测，而不是onPause？这是由于A启动B时，生命周期的执行顺序如下：A.onPause->B.onCreate->B.onStart->B.onResume->A.onStop，也就是说，在A的onPause方法中，B的生命周期还没有执行，进程没有进入前台，当然是检测不到的。我们把代码移到onPause生命周期中，发现确实没有效果。

       具体实现代码如下：
	  //用来控制应用前后台切换的逻辑  
	  private boolean isCurrentRunningForeground = true;  
	  @Override  
	  protected void onStart() {  
	      super.onStart();  
	      if (!isCurrentRunningForeground) {  
	          Log.d(TAG, ">>>>>>>>>>>>>>>>>>>切到前台 activity process");  
	      }  
	  }  
	  
	  @Override  
	  protected void onStop() {  
	      super.onStop();  
	      isCurrentRunningForeground = isRunningForeground();  
	      if (!isCurrentRunningForeground) {  
	          Log.d(TAG,">>>>>>>>>>>>>>>>>>>切到后台 activity process");  
	      }  
	  }  
	  
	  public boolean isRunningForeground() {  
	      ActivityManager activityManager = (ActivityManager) this.getSystemService(Context.ACTIVITY_SERVICE);  
	      List<ActivityManager.RunningAppProcessInfo> appProcessInfos = activityManager.getRunningAppProcesses();  
	      // 枚举进程  
	      for (ActivityManager.RunningAppProcessInfo appProcessInfo : appProcessInfos) {  
	          if (appProcessInfo.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {  
	              if (appProcessInfo.processName.equals(this.getApplicationInfo().processName)) {  
	                  Log.d(TAG,"EntryActivity isRunningForeGround");  
	                  return true;  
	              }  
	          }  
	      }  
	      Log.d(TAG, "EntryActivity isRunningBackGround");  
	      return false;  
	  }  