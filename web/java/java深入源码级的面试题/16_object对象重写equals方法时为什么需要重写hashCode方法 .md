
## object对象重写equals方法时为什么需要重写hashCode方法

### 参考答案

在Java语言中，equals方法在使用时：

    针对包装对象，比较的是对象的值（包括 boolean，byte，char，short，int，long，float，double）

    针对String对象，比较的也是String的值（因为String内部重写了equals方法和hashCode方法）

    针对其他object对象，比较的是两个对象的引用是否指向同一个内存地址


而当我们重写object对象的equals方法时，同时也要重写hashCode方法，为什么呢？ 首先来看几个定义


hash哈希又称之为散列，用于以常数平均时间执行插入、删除和查找的技术。（对排序不能得到有效的支持）

在对象存储的散列表里面，hashCode用来指定对象存储的内存地址。而equals用来判断对象的引用是否指向同一个地址，也就是判断两个对象的hashCode值是否一致



一般有约定

hashCode相同时，equals方法一定返回true。

equals方法返回true时，hashCode一定相同。


举个例子

    public class Main {  
          
        public static void main(String[] args) {  
            Str A = new Str(1);  
            Str B = new Str(1);  
            System.out.println("对比equals：" + A.equals(B));  //true  
            System.out.println("对比hashCode值" + (A.hashCode() == B.hashCode()));  //false  
              
            Set<Str> temp = new HashSet<Str>();  
            System.out.println("添加A到HashSet中：" + temp.add(A));  //true  
            System.out.println("添加B到HashSet中：" + temp.add(B));  //true  
        }  
          
        private static class Str{  
            public int val = 0;  
            public Str(int val){  
                this.val = val;  
            }  
            public boolean equals(Str str){  
                return val == str.val;  
            }  
        }  
    }  


如上，重写了Str类的equals方法，却没有重写它的hashCode方法，这就造成了混乱。

对象A和B的equals方法返回true

但是A和B的hashCode方法返回false


将两个对象添加到HashSet中的时候，由于HashSet默认对比的是hashCode值是否一致，A和B的hashCode值不一致，所以同时添加成功了，这样看，HashSet中就添加进去了两个equals返回true的对象，这和set的设计理念有误差，这就使得系统混乱了。

这也就是说，明明这两个对象是相等的，但是hashCode值不一样，所以当把A添加到HashSet中，而将B作为key去HashSet中查找时，根本查不到A。而A和B明明又是相等的，从而出现错误。给人的直观印象就是：“我要找的明明就是那个相等的对象，它的确也在集合里，但就是查不到”


关于String的hashCode方法

要注意的是，String重写了hashCode方法。这是因为散列表（hash表）操作中费时多的部分就是计算hashCode方法，所以在String类中的hashCode方法包含一个重要的优化：每个String对象内部都存储了它的hashCode值，该值初始为0，但如果hashCode方法被调用，那么这个值就将会被记住，下一次使用的时候可以直接调用出来，而不用再计算一次。之所以能这样实现，是由于String类是不可改变的。所以hashCode值被计算之后也并不会发生变化。

String类的hashCode的实现，简要摘录如下：

    public final class String{  
          
        private int hash = 0;  
          
        public int hashCode(){  
            if(hash != 0){  
                return hash;  
            }  
              
            for(int i=0; i<length(); i++){  
                hash = hash * 31 + (int) charAt(i);  
            }  
            return hash;  
        }  
    }  