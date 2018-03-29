
## Java异常体系

### 参考答案

[Java异常体系结构](https://blog.csdn.net/junlixxu/article/details/6096266)

java异常体系

|——Throwable 实现类描述java的错误和异常 一般交由硬件处理

　　　　|——Error(错误)一般不通过代码去处理，一般由硬件保护

　　　　|——Exception(异常)

　　　　　　|——RuntimeException(运行时异常)

　　　　　　|——非运行时异常
多个try-catch语句联用时的顺序

　　1、顺序执行，从上到下，有一个catch子句匹配之后，后面的自动不在执行

　　2、如果多个cach内的异常有父子类的关系

　　　　一定要，子类异常在上，父类异常在下

自定义异常类型

　　一般都是提供两个构造参数，一个无参一个有参数，有参数的一般是调用父类的有参构造函数，调用形式super(message)
运行时异常

RuntimeException

　　|——ClassCastException多态中可以使用instanceof 进行规避

　　|——ArithmeticException进行if判断，吐过除数为0进行return

　　|——NullPointerException进行if判断是否为null

　　|——ArrayIndexOutBondsExcetion使用数组length属性以避免数组越界。

　　在后面我们异常处理的时候，经常把捕获的一场装华为运行时异常抛出，尤其是写一些函数框架时。throw new RuntimeException(e);

非运行时异(受检异常) 这些异常必须做出try-catch不然编译器无法通过 注意事项

　　1、子类覆盖父类的方法时，父类方法抛出异常，子类的覆盖方法可以不抛出异常或者抛出父类方法相同的异常，或者抛出父类方法异常的子类。

　　2、父类方法抛出了多个异常，子类覆盖方法时，只能抛出父类异常的子集

　　3、父类没有抛出异常，子类不能抛出异常。子类发生非运行时异常时，需要进行try-catch处理

　　4、子类不能比父类抛出更多的异常。

　　凡事应当向父类看齐，父类已有就应当向分类看齐。
 
finally块 一般用于释放资源 无论程序正常与否都执行finally块

　　1.只有一种情况，jvm退出了System.exit(0)这时候不会执行finally的内容

　　2、return语句也无法阻止finally的执行

