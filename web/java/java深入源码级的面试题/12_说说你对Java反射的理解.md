
## 谈谈Java反射机制

### 参考答案

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

JAVA反射（放射）机制：“程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言”。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。但是JAVA有着一个非常突出的动态相关机制：Reflection，用在Java身上指的是我们可以于运行时加载、探知、使用编译期间完全未知的classes。换句话说，Java程序可以加载一个运行时才得知名称的class，获悉其完整构造（但不包括methods定义），并生成其对象实体、或对其fields设值、或唤起其methods。

Java反射机制主要提供了以下功能： 在运行时判断任意一个对象所属的类；在运行时构造任意一个类的对象；在运行时判断任意一个类所具有的成员变量和方法；在运行时调用任意一个对象的方法；生成动态代理。

利用反射机制能获得什么信息

一句话，类中有什么信息，它就可以获得什么信息，不过前提是得知道类的名字，要不就没有后文了

首先得根据传入的类的全名来创建Class对象。

	Class c=Class.forName("className");注明：className必须为全名，也就是得包含包名，比如，cn.netjava.pojo.UserInfo;
	Object obj=c.newInstance();//创建对象的实例
	OK，有了对象就什么都好办了，想要什么信息就有什么信息了。  
	获得构造函数的方法
    Constructor getConstructor(Class[] params)//根据指定参数获得public构造器

    Constructor[] getConstructors()//获得public的所有构造器

    Constructor getDeclaredConstructor(Class[] params)//根据指定参数获得public和非public的构造器

    Constructor[] getDeclaredConstructors()//获得public的所有构造器
   
    获得类方法的方法
    Method getMethod(String name, Class[] params),根据方法名，参数类型获得方法

    Method[] getMethods()//获得所有的public方法

    Method getDeclaredMethod(String name, Class[] params)//根据方法名和参数类型，获得public和非public的方法

    Method[] getDeclaredMethods()//获得所以的public和非public方法
   
    获得类中属性的方法
    Field getField(String name)//根据变量名得到相应的public变量

    Field[] getFields()//获得类中所以public的方法

    Field getDeclaredField(String name)//根据方法名获得public和非public变量

    Field[] getDeclaredFields()//获得类中所有的public和非public方法

Java的反射非常强大，传递class， 可以动态的生成该类、取得这个类的所有信息，包括里面的属性、方法以及构造函数等，甚至可以取得其父类或父接口里面的内容。
  obj.getClass().getDeclaredMethods();//取得obj类中自己定义的方法， 包括私有的方法。
  obj.getClass().getMethods();//取得obj类中自己定义的方法及继承过来的方法， 但私有方法得不到。
  同样， 对field也是一样，obj.getClass().getDeclaredFields();取得的是这个类中所有的属性，包括私有的field; 对obj.getClass().getFields();//取得是自己以及接继承来的属性， 但不能取得自己的私有属性。



    static Object create(Class clazz) throws Exception {  
        Constructor con = clazz.getConstructor(String.class);  
        Object obj = con.newInstance("test name");  
        return obj;  
      }  
      
      static void invoke1(Object obj, String methodName)  
          throws IllegalArgumentException, IllegalAccessException,  
          InvocationTargetException, Exception, NoSuchMethodException {  
        Method[] ms = obj.getClass().getDeclaredMethods();  
        ms = obj.getClass().getMethods();  
        for (Method m : ms) {  
          // System.out.println(m.getName());  
          if (methodName.equals(m.getName()))  
            m.invoke(obj, null);  
        }  
      
        Method m = obj.getClass().getMethod(methodName, null);  
        m.invoke(obj, null);  
      }  
      
      static void field(Class clazz) throws Exception {  
        Field[] fs = clazz.getDeclaredFields();  
        //fs = clazz.getFields();  
        for (Field f : fs)  
          System.out.println(f.getName());  
      }  
        
      static void annon(Class clazz) throws Exception {  
        Annotation[] as = clazz.getAnnotations();  
      }  
