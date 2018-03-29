
## Object类到底实现了哪些方法？

### 答案

1. clone方法

	保护方法，实现对象的浅复制，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。

2. getClass方法
	
	final方法，获得运行时类型。

3. toString方法
	
	该方法用得比较多，一般子类都有覆盖。
	
4. finalize方法
	
	该方法用于释放资源。因为无法确定该方法什么时候被调用，很少使用。
	
5. equals方法
	
	该方法是非常重要的一个方法。一般equals和==是不一样的，但是在Object中两者是一样的。子类一般都要重写这个方法。
	
6. hashCode方法
	
	该方法用于哈希查找，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。
	
	一般必须满足obj1.equals(obj2)==true。可以推出obj1.hash-Code()==obj2.hashCode()，但是hashCode相等不一定就满足equals。不过为了提高效率，应该尽量使上面两个条件接近等价。
	
7. wait方法
	
	wait方法就是使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。wait()方法一直等待，直到获得锁或者被中断。wait(longtimeout)设定一个超时间隔，如果在规定时间内没有获得锁就返回。
	
	调用该方法后当前线程进入睡眠状态，直到以下事件发生。
	
	（1）其他线程调用了该对象的notify方法。
	
	（2）其他线程调用了该对象的notifyAll方法。
	
	（3）其他线程调用了interrupt中断该线程。
	
	（4）时间间隔到了。
	
	此时该线程就可以被调度了，如果是被中断的话就抛出一个InterruptedException异常。
	
8. notify方法
	
	该方法唤醒在该对象上等待的某个线程。
	
9. notifyAll方法
	
	该方法唤醒在该对象上等待的所有线程。

### 答案

* protected  Object clone()    
  创建并返回此对象的一个副本。 
* boolean equals(Object obj)    
  指示某个其他对象是否与此对象“相等”。 
* protected  void finalize()    
  当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。 
* Class<?  extendsObject> getClass()      
  返回一个对象的运行时类。 
* int hashCode()       
  返回该对象的哈希码值。 
* void notify()        
  唤醒在此对象监视器上等待的单个线程。 
* void notifyAll()     
  唤醒在此对象监视器上等待的所有线程。 
* String toString()          
  返回该对象的字符串表示。 
* void wait()     
  导致当前的线程等待，直到其他线程调用此对象的notify() 方法或 notifyAll() 方法。 
* void wait(long timeout)  
  导致当前的线程等待，直到其他线程调用此对象的notify() 方法或 notifyAll() 方法，或者超过指定的时间量。 
* void wait(long timeout,  int nanos)  
  导致当前的线程等待，直到其他线程调用此对象的notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。 
