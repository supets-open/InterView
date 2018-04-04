Activity与Service之间通信

Activity启动服务Service时，通过Intent启动。若Activity向Service传递参数可以通过Intent传递。而当Service某些操作更新UI时，我们该如何操作呢？启动Service又有startService和bindService两种方式，不同的启动方式又该如何通信?以下有几种通信方式：

1 在Activity中，通过bindService方式启动Service，获取Service实例，然后进行操作

* 在Activity中
 
	ServiceConnection conn = new ServiceConnection() {  
	            @Override  
	            public void onServiceConnected(ComponentName name, IBinder service) {  
	                MyService.MyBinder mMyBinder = (MyService.MyBinder) service;  
	                // 获取Service对象  
	                mMyService =  mMyBinder.getService();  
	            }  
	    }  
	    // 通过bindService绑定Service  
	    Intent intentBind = new Intent(this, MyService_.class);  
	        bindService(intentBind, conn, BIND_AUTO_CREATE);

* 在Service中

	public IBinder onBind(Intent intent) {  
	          
	        return new MyBinder();  
	    }  
	    // 创建内部类MyBinder继承于Binder,以便获取Service实例  
	    public class MyBinder extends Binder {  
	        public MyService getService() {  
	            return MyService.this;  
	        }  
	    }

2 通过回调的方式。如方法1，已经获取Service实例。在Actvity中实现接口，在Service中，设置接口实例并调用。

* 在Activity中

	ServiceConnection conn = new ServiceConnection() {  
	    
		@Override  
	   public void onServiceConnected(ComponentName name, IBinder service) {  
	      MyService.MyBinder mMyBinder = (MyService.MyBinder) service;  
	      mMyService =  mMyBinder.getService();  
	  
	      // 通过匿名内部类实例化接口  
	      mMyService.setListener(new MyService.onProgressChangedListener() {  
	           @Override  
	           public void setProgress(int progress) {  
	                          
	           }  
	        });  
	}

* 在Service中

	public class MyService extends Service {  
	          
	        ***...  
	  
	        onProgressChangedListener mListener;  
	          
	        // 定义接口  
	        public interface onProgressChangedListener {  
	            void setProgress(int progress);  
	        }  
	  
	        ***...  
	        // 获取接口实例  
	        public void setListener(onProgressChangedListener listener) {  
	            this.mListener = listener;  
	        }  
	  
	        ***...  
	    }

3 通过广播BroadcastReiceive通信。

通过starService启动Service时，并不能够获取Service，意味着方式1和2，都不可再实现。我们可以这样做，在Service需要更新UI时，发送一条广播。在Activity中注册广播并接受广播处理。此种方式，只需要发送广播，并不需要获取Service实例,那么问题迎刃而解。

在stopService或unBindService时，需要调用unregisterReceiver，将广播注销掉。