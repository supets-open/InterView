[LocalBroadcastManager 的实现原理，还是 Binder？](http://www.trinea.cn/android/localbroadcastmanager-impl/)



对 LocalBroadcastManager 大家应该都不陌生，相对 BroadcastReceiver，它只能用于应用内通信，安全性更好，同时拥有更高的运行效率。也是需要发送应用内广播时的官方推荐。

  

大家也都知道BroadcastReceiver的通信是走 Binder 机制的，而 LocalBroadcastManager 因为叫LocalBroadcast，可能让人产生一种它也是以 Binder 通讯方式为底层实现的错觉，点进源码，我们会发现这个更安全高效的实现原来如此熟悉。

 

还是先简单提下 LocalBroadcastManager 使用，更多可见：BroadcastReceiver 详细介绍。

  

1. LocalBroadcastManager 使用

LocalBroadcastManager 的使用跟一般 BroadcastReceiver 差别不大。

(1) 自定义 BroadcastReceiver 子类

 


Java

public class LocalBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        localMsg.setText(intent.getStringExtra(MSG_KEY));
    }
}

 public class LocalBroadcastReceiver extends BroadcastReceiver {
 
    @Override
    public void onReceive(Context context, Intent intent) {
        localMsg.setText(intent.getStringExtra(MSG_KEY));
    }
}

  

(2) 注册接收器

 


Java

LocalBroadcastReceiver localReceiver = new LocalBroadcastReceiver();
LocalBroadcastManager.getInstance(context).registerReceiver(localReceiver, new IntentFilter(ACTION_LOCAL_SEND));


 LocalBroadcastReceiver localReceiver = new LocalBroadcastReceiver();
LocalBroadcastManager.getInstance(context).registerReceiver(localReceiver, new IntentFilter(ACTION_LOCAL_SEND));

 
 

  

(3) 发送广播

 

 



Java

LocalBroadcastManager.getInstance(context).sendBroadcast(new Intent(ACTION_LOCAL_SEND));

 LocalBroadcastManager.getInstance(context).sendBroadcast(new Intent(ACTION_LOCAL_SEND));

  

(4) 取消注册


Java

LocalBroadcastManager.getInstance(context).unregisterReceiver(localReceiver);





1
 LocalBroadcastManager.getInstance(context).unregisterReceiver(localReceiver);


2. 实现

LocalBroadcastManager 源代码可见：LocalBroadcastManager.java

(1) 构造函数


Java

public static LocalBroadcastManager getInstance(Context context) {
    synchronized (mLock) {
        if (mInstance == null) {
            mInstance = new LocalBroadcastManager(context.getApplicationContext());
        }
        return mInstance;
    }
}

private LocalBroadcastManager(Context context) {
    mAppContext = context;
    mHandler = new Handler(context.getMainLooper()) {

        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_EXEC_PENDING_BROADCASTS:
                    executePendingBroadcasts();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    };
}

 public static LocalBroadcastManager getInstance(Context context) {
    synchronized (mLock) {
        if (mInstance == null) {
            mInstance = new LocalBroadcastManager(context.getApplicationContext());
        }
        return mInstance;
    }
}
 
private LocalBroadcastManager(Context context) {
    mAppContext = context;
    mHandler = new Handler(context.getMainLooper()) {
 
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_EXEC_PENDING_BROADCASTS:
                    executePendingBroadcasts();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    };
}

 
 

先看构造函数，单例实现因而私有化构造函数。
 注意的是基于主线程的 Looper 新建了一个 Handler，handleMessage中会调用接收器对广播的消息进行处理，也是 LocalBroadcastManager 的核心部分，具体见后面executePendingBroadcasts()介绍。

  

单例函数还可以通过双层条件判断提高效率，双层条件判断的写法可见：单例模式

 

(2) 注册接收器

Java

HashMap<BroadcastReceiver, ArrayList<IntentFilter>> mReceivers
            = new HashMap<BroadcastReceiver, ArrayList<IntentFilter>>();
HashMap<String, ArrayList<ReceiverRecord>> mActions
            = new HashMap<String, ArrayList<ReceiverRecord>>();

public void registerReceiver(BroadcastReceiver receiver, IntentFilter filter) {
    synchronized (mReceivers) {
        ReceiverRecord entry = new ReceiverRecord(filter, receiver);
        ArrayList<IntentFilter> filters = mReceivers.get(receiver);
        if (filters == null) {
            filters = new ArrayList<IntentFilter>(1);
            mReceivers.put(receiver, filters);
        }
        filters.add(filter);
        for (int i=0; i<filter.countActions(); i++) {
            String action = filter.getAction(i);
            ArrayList<ReceiverRecord> entries = mActions.get(action);
            if (entries == null) {
                entries = new ArrayList<ReceiverRecord>(1);
                mActions.put(action, entries);
            }
            entries.add(entry);
        }
    }
}  


 HashMap<BroadcastReceiver, ArrayList<IntentFilter>> mReceivers
            = new HashMap<BroadcastReceiver, ArrayList<IntentFilter>>();
HashMap<String, ArrayList<ReceiverRecord>> mActions
            = new HashMap<String, ArrayList<ReceiverRecord>>();
 
public void registerReceiver(BroadcastReceiver receiver, IntentFilter filter) {
    synchronized (mReceivers) {
        ReceiverRecord entry = new ReceiverRecord(filter, receiver);
        ArrayList<IntentFilter> filters = mReceivers.get(receiver);
        if (filters == null) {
            filters = new ArrayList<IntentFilter>(1);
            mReceivers.put(receiver, filters);
        }
        filters.add(filter);
        for (int i=0; i<filter.countActions(); i++) {
            String action = filter.getAction(i);
            ArrayList<ReceiverRecord> entries = mActions.get(action);
            if (entries == null) {
                entries = new ArrayList<ReceiverRecord>(1);
                mActions.put(action, entries);
            }
            entries.add(entry);
        }
    }
}  

 
 

mReceivers 存储广播和过滤器信息，以BroadcastReceiver作为 key，IntentFilter链表作为 value。
mReceivers 是接收器和IntentFilter的对应表，主要作用是方便在unregisterReceiver(…)取消注册，同时作为对象锁限制注册接收器、发送广播、取消接收器注册等几个过程的并发访问。

 

mActions 以Action为 key，注册这个Action的BroadcastReceiver链表为 value。mActions 的主要作用是方便在广播发送后快速得到可以接收它的BroadcastReceiver。

 

(3) 发送广播



Java

public boolean sendBroadcast(Intent intent) {
    synchronized (mReceivers) {
        final String action = intent.getAction();
        final String type = intent.resolveTypeIfNeeded(mAppContext.getContentResolver());
        final Uri data = intent.getData();
        final String scheme = intent.getScheme();
        final Set<String> categories = intent.getCategories();
        ……
        ArrayList<ReceiverRecord> entries = mActions.get(intent.getAction());
        if (entries != null) {
            if (debug) Log.v(TAG, "Action list: " + entries);

            ArrayList<ReceiverRecord> receivers = null;
            for (int i=0; i<entries.size(); i++) {
                ReceiverRecord receiver = entries.get(i);
                if (receiver.broadcasting) {
                    if (debug) {
                        Log.v(TAG, "  Filter's target already added");
                    }
                    continue;
                }

                int match = receiver.filter.match(action, type, scheme, data,
                        categories, "LocalBroadcastManager");
                if (match >= 0) {
                    if (debug) Log.v(TAG, "  Filter matched!  match=0x" +
                            Integer.toHexString(match));
                    if (receivers == null) {
                        receivers = new ArrayList<ReceiverRecord>();
                    }
                    receivers.add(receiver);
                    receiver.broadcasting = true;
                } else {
                    ……
                }
            }

            if (receivers != null) {
                for (int i=0; i<receivers.size(); i++) {
                    receivers.get(i).broadcasting = false;
                }
                mPendingBroadcasts.add(new BroadcastRecord(intent, receivers));
                if (!mHandler.hasMessages(MSG_EXEC_PENDING_BROADCASTS)) {
                    mHandler.sendEmptyMessage(MSG_EXEC_PENDING_BROADCASTS);
                }
                return true;
            }
        }
    }
    return false;
}


 public boolean sendBroadcast(Intent intent) {
    synchronized (mReceivers) {
        final String action = intent.getAction();
        final String type = intent.resolveTypeIfNeeded(mAppContext.getContentResolver());
        final Uri data = intent.getData();
        final String scheme = intent.getScheme();
        final Set<String> categories = intent.getCategories();
        ……
        ArrayList<ReceiverRecord> entries = mActions.get(intent.getAction());
        if (entries != null) {
            if (debug) Log.v(TAG, "Action list: " + entries);
 
            ArrayList<ReceiverRecord> receivers = null;
            for (int i=0; i<entries.size(); i++) {
                ReceiverRecord receiver = entries.get(i);
                if (receiver.broadcasting) {
                    if (debug) {
                        Log.v(TAG, "  Filter's target already added");
                    }
                    continue;
                }
 
                int match = receiver.filter.match(action, type, scheme, data,
                        categories, "LocalBroadcastManager");
                if (match >= 0) {
                    if (debug) Log.v(TAG, "  Filter matched!  match=0x" +
                            Integer.toHexString(match));
                    if (receivers == null) {
                        receivers = new ArrayList<ReceiverRecord>();
                    }
                    receivers.add(receiver);
                    receiver.broadcasting = true;
                } else {
                    ……
                }
            }
 
            if (receivers != null) {
                for (int i=0; i<receivers.size(); i++) {
                    receivers.get(i).broadcasting = false;
                }
                mPendingBroadcasts.add(new BroadcastRecord(intent, receivers));
                if (!mHandler.hasMessages(MSG_EXEC_PENDING_BROADCASTS)) {
                    mHandler.sendEmptyMessage(MSG_EXEC_PENDING_BROADCASTS);
                }
                return true;
            }
        }
    }
    return false;
}

 
 

先根据Action从mActions中取出ReceiverRecord列表，循环每个ReceiverRecord判断 filter 和 intent 中的 action、type、scheme、data、categoried 是否 match，是的话则保存到receivers列表中，发送 what 为MSG_EXEC_PENDING_BROADCASTS的消息，通过 Handler 去处理。

  

关于 match 规则可见：Intent Filter介绍。

 

(4) 消息处理


Java

private void executePendingBroadcasts() {
    while (true) {
        BroadcastRecord[] brs = null;
        synchronized (mReceivers) {
            final int N = mPendingBroadcasts.size();
            if (N <= 0) {
                return;
            }
            brs = new BroadcastRecord[N];
            mPendingBroadcasts.toArray(brs);
            mPendingBroadcasts.clear();
        }
        for (int i=0; i<brs.length; i++) {
            BroadcastRecord br = brs[i];
            for (int j=0; j<br.receivers.size(); j++) {
                br.receivers.get(j).receiver.onReceive(mAppContext, br.intent);
            }
        }
    }
}

 private void executePendingBroadcasts() {
    while (true) {
        BroadcastRecord[] brs = null;
        synchronized (mReceivers) {
            final int N = mPendingBroadcasts.size();
            if (N <= 0) {
                return;
            }
            brs = new BroadcastRecord[N];
            mPendingBroadcasts.toArray(brs);
            mPendingBroadcasts.clear();
        }
        for (int i=0; i<brs.length; i++) {
            BroadcastRecord br = brs[i];
            for (int j=0; j<br.receivers.size(); j++) {
                br.receivers.get(j).receiver.onReceive(mAppContext, br.intent);
            }
        }
    }
}

 
 

以上为消息处理的函数。mPendingBroadcasts转换为数组BroadcastRecord，循环每个receiver，调用其onReceive函数，这样便完成了广播的核心逻辑。

 

(5) 取消注册

 

 



Java

public void unregisterReceiver(BroadcastReceiver receiver) {
    synchronized (mReceivers) {
        ArrayList<IntentFilter> filters = mReceivers.remove(receiver);
        if (filters == null) {
            return;
        }
        for (int i=0; i<filters.size(); i++) {
            IntentFilter filter = filters.get(i);
            for (int j=0; j<filter.countActions(); j++) {
                String action = filter.getAction(j);
                ArrayList<ReceiverRecord> receivers = mActions.get(action);
                if (receivers != null) {
                    for (int k=0; k<receivers.size(); k++) {
                        if (receivers.get(k).receiver == receiver) {
                            receivers.remove(k);
                            k--;
                        }
                    }
                    if (receivers.size() <= 0) {
                        mActions.remove(action);
                    }
                }
            }
        }
    }
}



 public void unregisterReceiver(BroadcastReceiver receiver) {
    synchronized (mReceivers) {
        ArrayList<IntentFilter> filters = mReceivers.remove(receiver);
        if (filters == null) {
            return;
        }
        for (int i=0; i<filters.size(); i++) {
            IntentFilter filter = filters.get(i);
            for (int j=0; j<filter.countActions(); j++) {
                String action = filter.getAction(j);
                ArrayList<ReceiverRecord> receivers = mActions.get(action);
                if (receivers != null) {
                    for (int k=0; k<receivers.size(); k++) {
                        if (receivers.get(k).receiver == receiver) {
                            receivers.remove(k);
                            k--;
                        }
                    }
                    if (receivers.size() <= 0) {
                        mActions.remove(action);
                    }
                }
            }
        }
    }
}

 
 

从mReceivers及mActions中移除相应元素。

 

到此为止我们便非常清晰了：
(1) LocalBroadcastManager 的核心实现实际还是 Handler，只是利用到了 IntentFilter 的 match 功能，至于 BroadcastReceiver 换成其他接口也无所谓，顺便利用了现成的类和概念而已。
(2) 因为是 Handler 实现的应用内的通信，自然安全性更好，效率更高。
