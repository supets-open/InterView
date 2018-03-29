
## 说说你对Java注解的理解

### 参考答案

[秒懂，Java 注解 （Annotation）你可以这样学](https://blog.csdn.net/briblue/article/details/73824058)

注解的功能分为2部分：

    1 作为特定的标记

    2 额外信息的载体

注解分类：
1 元注解

@Target,@Retention,@Documented,@Inherited

大致可分为三类：
编写文档：通过代码里标识的元数据生成文档。
代码分析：通过代码里标识的元数据对代码进行分析。
编译检查：通过代码里标识的元数据让编译器能实现基本的编译检查


2 基本内置注解

@Override
它的作用是对覆盖超类中方法的方法进行标记，如果被标记的方法并没有实际覆盖超类中的方法，则编译器会发出错误警告。

@Deprecated
它的作用是对不应该再使用的方法添加注解，当编程人员使用这些方法时，将会在编译时显示提示信息，它与javadoc里的@deprecated标记有相同的功能，准确的说，它还不如javadoc @deprecated，因为它不支持参数

@SuppressWarnings
其参数有：
deprecation，使用了过时的类或方法时的警告
unchecked，执行了未检查的转换时的警告
fallthrough，当 switch 程序块直接通往下一种情况而没有 break 时的警告
path，在类路径、源文件路径等中有不存在的路径时的警告
serial，当在可序列化的类上缺少serialVersionUID 定义时的警告
finally ，任何 finally 子句不能正常完成时的警告
all，关于以上所有情况的警告

3 自定义注解

它类似于新创建一个接口文件，但为了区分，我们需要将它声明为@interface,如下例：

public @interface NewAnnotation {
}
使用自定义的注解类型

public class AnnotationTest {
    @NewAnnotation
    public static void main(String[]args) {
    }
}
为自定义注解添加变量

Java代码
public @interface NewAnnotation {
    String value();
}


四个元注解分别是：@Target,@Retention,@Documented,@Inherited ，再次强调下元注解是java API提供，是专门用来定义注解的注解，其作用分别如下。
  
        @Target 表示该注解用于什么地方，可能的值在枚举类 ElemenetType 中，包括：
          ElemenetType.CONSTRUCTOR 构造器声明
          ElemenetType.FIELD 域声明（包括 enum 实例）

          ElemenetType.LOCAL_VARIABLE 局部变量声明
          ElemenetType.ANNOTATION_TYPE 作用于注解量声明

          ElemenetType.METHOD 方法声明
          ElemenetType.PACKAGE 包声明
          ElemenetType.PARAMETER 参数声明
          ElemenetType.TYPE 类，接口（包括注解类型）或enum声明
          
     @Retention 表示在什么级别保存该注解信息。可选的参数值在枚举类型 RetentionPolicy 中，包括：
          RetentionPolicy.SOURCE 注解将被编译器丢弃
          RetentionPolicy.CLASS 注解在class文件中可用，但会被VM丢弃
          RetentionPolicy.RUNTIME VM将在运行期也保留注释，因此可以通过反射机制读取注解的信息。
          
      @Documented 将此注解包含在 javadoc 中 ，它代表着此注解会被javadoc工具提取成文档。在doc文档中的内容会因为此注解的信息内容不同而不同。相当与@see,@param 等。
      
      @Inherited 允许子类继承父类中的注解，例子中补充。

注解高级应用
编辑
使用范围
用@Target指定ElementType属性
	
Java代码(jdk)
public enum ElementType {
    // 用于类，接口，枚举但不能是注解
    TYPE,
    // 字段上，包括枚举值
    FIELD,
    // 方法，不包括构造方法
    METHOD,
    // 方法的参数
    PARAMETER,
    // 构造方法
    CONSTRUCTOR,
    // 本地变量或catch语句
    LOCAL_VARIABLE,
    // 注解类型(无数据)
    ANNOTATION_TYPE,
    // Java包
    PACKAGE
}
	
Java代码
// 限制注解使用范围
@Target({ElementType.METHOD,ElementType.CONSTRUCTOR})
public @interface Greeting {
    // 使用枚举类型
    public enum FontColor {
        BLUE,RED,GREEN
    };
    String name();
    FontColor fontColor() default FontColor.RED;
}
注解保持性策略

在Java编译器编译时，它会识别在源代码里添加的注解是否还会保留，这就是RetentionPolicy。下面是Java定义的RetentionPolicy枚举：
编译器的处理有三种策略：
将注解保留在编译后的类文件中，并在第一次加载类时读取它；
将注解保留在编译后的类文件中，但是在运行时忽略它；
按照规定使用注解，但是并不将它保留到编译后的类文件中。

public enum RetentionPolicy {
    // 此类型会被编译器丢弃
    SOURCE,
    // 此类型注解会保留在class文件中，但JVM会忽略它
    CLASS,
    // 此类型注解会保留在class文件中，JVM会读取它
    RUNTIME
}
	
Java代码
// 让保持性策略为运行时态，即将注解编码到class文件中，让虚拟机读取
@Retention(RetentionPolicy.RUNTIME)
public @interface Greeting {
    // 使用枚举类型
    public enum FontColor {
        BLUE,RED,GREEN
    };
    String name();
    FontColor fontColor() default FontColor.RED;
}
文档化功能
Java提供的Documented元注解跟Javadoc的作用是差不多的，其实它存在的好处是开发人员可以定制Javadoc不支持的文档属性，并在开发中应用。它的使用跟前两个也是一样的，简单代码示例如下：
	
Java代码
// 让它定制文档化功能
// 使用此注解时必须设置RetentionPolicy为RUNTIME
@Documented
public @interface Greeting {
    // 使用枚举类型
    public enum FontColor {
        BLUE,RED,GREEN
    };
    String name();
    FontColor fontColor() default FontColor.RED;
}
标注继承

Java代码
// 让它允许继承，可作用到子类
@Inherited
public @interface Greeting {
    // 使用枚举类型
    public enum FontColor {
        BLUE,RED,GREEN
    };
    String name();
    FontColor fontColor() default FontColor.RED;
}
读取方法
编辑
属于重点，在系统中用到注解权限时非常有用，可以精确控制权限的粒度
注意：要想使用反射去读取注解，必须将Retention的值选为Runtime

 
//读取注解信息
public class ReadAnnotationInfoTest {
    public static void main(String[] args) throws Exception {
        // 测试AnnotationTest类，得到此类的类对象
        Class c = Class.forName("com.iwtxokhtd.annotation.AnnotationTest");
        // 获取该类所有声明的方法
        Method[] methods = c.getDeclaredMethods();
        // 声明注解集合
        Annotation[] annotations;
        // 遍历所有的方法得到各方法上面的注解信息
        for (Method method : methods) {
            // 获取每个方法上面所声明的所有注解信息
            annotations = method.getDeclaredAnnotations();
            // 再遍历所有的注解，打印其基本信息
            System.out.println(method.getName());
            for (Annotation an : annotations) {
                System.out.println("方法名为：" + method.getName() + "其上面的注解为：" + an.annotationType().getSimpleName());
                Method[] meths = an.annotationType().getDeclaredMethods();
                // 遍历每个注解的所有变量
                for (Method meth : meths) {
                    System.out.println("注解的变量名为：" + meth.getName());
                }
            }
        }
    }
}