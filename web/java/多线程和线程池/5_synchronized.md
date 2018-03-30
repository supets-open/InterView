
19、谈谈对Synchronized关键字，类锁，方法锁，重入锁的理解
[synchronized的功能拓展:重入锁(读书笔记)](https://www.cnblogs.com/ten951/p/6212118.html)

[java synchronized关键字，类锁，对象锁详解](https://blog.csdn.net/su_xiaocheng/article/details/51859673)

20、static synchronized 方法的多线程访问和作用
[Java多线程中Synchronized简介和Static Synchronized的区别](https://blog.csdn.net/lulei1217/article/details/50954267)


synchronized是对当前的实例进行加锁，要注意是“当前实例”，也就是说，假如你有两个实例化对象，那么可以同时访问这两个实例里面的synchronized块。但是，当访问一个实例里面的一个synchronized块时，其余的synchronized是不可同时访问的，原因是整个实例都被加了锁。

那假如说是static synchronized时会怎么样呢？ 那么static synchronized恰好就是要控制类的所有实例的访问了，static synchronized是限制线程同时访问jvm中该类的所有实例同时访问对应的代码块。实际上，假如类中某一个方法或者说某个代码块是synchronized的话，那在实例化时就会自动生成一个监视块，用于不同线程之间访问的互斥。而static synchronized则是所有该类的实例公用一个监视块了。

	　　有一个例子：
	　　pulbic class Something(){
	　　public synchronized void isSyncA(){}
	　　public synchronized void isSyncB(){}
	　　public static synchronized void cSyncA(){}
	　　public static synchronized void cSyncB(){}
	　　}
	　　那么，加入有Something类的两个实例a与b，那么下列组方法可以被1个以上线程同时访问呢？
	　　a.   x.isSyncA()与x.isSyncB()
	　　b.   x.isSyncA()与y.isSyncA()
	　　c.   x.cSyncA()与y.cSyncB()
	　　d.   x.isSyncA()与Something.cSyncA()
	　　a的话是同个实例中的，故不可以；b中是不同实例的，所以可以同时访问；c中是针对static synchronized，上文所说所有的static方法共用一个监视块，故不能够同时访问；那么d呢？实际上是可以的，原因可以自己思考一下。
	　　同时还有一点，synchronized关键字是不能够继承的。

21、同一个类里面两个synchronized方法，两个线程同时访问的问题
[Java中一个类里面有两个用synchronized修饰的非静态方法，不同的线程中的实例访问这两个方法时会发生什么](https://blog.csdn.net/Sunjy1881/article/details/76269829)

[关于java多线程中同步的问题(两个线程访问同一个实例类的两个同步方法,会不会互相影响)](https://blog.csdn.net/fycghy0803/article/details/2182127)

[Java中同一个类中不同的synchronized方法是否可以并发执行 ?](https://blog.csdn.net/niusi1288/article/details/76143276)
