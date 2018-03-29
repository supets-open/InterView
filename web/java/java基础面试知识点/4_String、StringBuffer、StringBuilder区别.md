
## String、StringBuffer、StringBuilder区别

### 答案

1、String，StingBuffer，StringBuilder是字符串变量还是常量？

 String ----> 字符串常量
 StringBuffer ----> 字符串变量（线程安全的）
 StringBuilder ----> 字符串变量（非线程安全的
 官方解释说：String 是一个常量，他的值在创建之后不能改变，字符串缓冲区支持修改，因为字符串对象是不可变的，所以支持共享。
 
 String s = "abc";  
 s = s + 1;  
 System.out.println(s);
 
 代码解析：首先我们定义一个变量s，给他赋值“abc”，然后在创建一个新的对象s用来执行s=s+1;也就是说我们之前的s并没有发生变化。这也说明了String类型是一个不可改变的对象，每次操作String字符串，实际上是不断的创建新的对象，而原来的对象变成了垃圾被GC回收，从而String字符串的效率也是最低的。
 
 String，StingBuffer，StringBuilder效率如何呢？
 
 从高到底的顺序依次是：StringBuilder  >  StingBuffer > String
 有上知道了String是字符串常量，所以他的效率自然而然是最低的。对于StringBuffer和StringBuilder它们属于变量，是可以改变的对象，每次对字符串的操作，实际上实在一个对象上操作，所以效率更高一些。StringBuffer是线程安全的，考虑到安全问题，相对他的性能会更低一点。由此知道了从效率的角度看，StringBuilder最高，其次是StringBuffer，最后是String字符串常量。
 
 总结：当多个线程使用字符串缓冲区时，使用StringBuffer保证正确的操作，如果是单线程的，建议使用StringBuiler效率更高一些。