启动Activity
运行的生命周期函数为：onCreate  onStart  onResume
按back键返回 
运行的生命周期函数为：onPause  onStop   onDestroy

按home键
运行的生命周期函数为：onPause  onStop
然后再次进入
运行的生命周期函数为：onRestart  onStart  onResume

finish方法
在Actyvity A 中的onResume方法中使用startActivity的方法进行开启另外一个Activity B，如：
protected void onResume() {  
        // TODO Auto-generated method stub  
        super.onResume();  
        Log.d("zmq","onResume()");  
        Intent intent = new Intent();  
        intent.setClass(MainActivity.this, TestActivity.class);  
        startActivity(intent);  
        finish();  
        Log.d("zmq","onResume() I hava run finish()");  
    }  

此时从Activity A开始运行到打开另外一个Activity B，Activity A中生命周期函数调用顺序：
onCreate
onStart
onResume
onPause
onStop
onDestroy

dialog对话框
在Activity A中启动一个对话框风格的Activity
Activity A运行的生命周期函数为：
onPause
点击返回键，Activity A运行的生命周期函数为：
onResume

打开一个一般的Activity，不做finish动作
此时从Activity A开始运行到打开另外一个Activity B，Activity A中生命周期函数调用顺序：
onCreate  onStart  onResume  onPause  onStop
然后按返回键退出，Activity A运行的生命周期函数为：onRestart  onStart  onResume