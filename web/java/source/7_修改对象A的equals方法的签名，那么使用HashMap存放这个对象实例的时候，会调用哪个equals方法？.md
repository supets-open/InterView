
## 修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法？

### 参考答案

会调用对象对象的equals方法。

“==”如果是基本类型的话就是看他们的数据值是否相等就可以。

如果是引用类型的话，比较的是栈内存局部变量表中指向堆内存中的指针的值是否相等
“equals”如果对象的equals方法没有重写的话，equals方法和“==”是同一种。

hashcod是返回对象实例内存地址的hash映射。
理论上所有对象的hash映射都是不相同的。

问题：

1.你用过hashmap么？

答：是的，然后回答HashMap的一些特性，譬如HashMap可以接受null键值和值，而Hashtable则不能；HashMap是非synchronized;HashMap很快；以及HashMap储存的是键值对等等。

2.你知道hashmap的工作原理么？

答：HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关键点在于指出，HashMap是在bucket中储存键对象和值对象，作为Map.Entry。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道hashing以及HashMap的工作原理。但是这仅仅是故事的开始，当面试官加入一些Java程序员每天要碰到的实际场景的时候，错误的答案频现。

for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
System.out.println(entry.getKey() + ":" + entry.getValue());
}

3.当两个对象的hashcode相同会发生什么？

因为即使hashcode相同，但是equals不同。
因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。

4.如果两个键的hashcode相同，你如何获取值对象？

当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。

5.面试官提醒他如果有两个值对象储存在同一个bucket？

将会遍历链表直到找到值对象

6.因为你并没有值对象去比较，你是如何确定确定找到值对象的？

找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象。

一些优秀的开发者会指出使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

7.如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？

默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

8.使用concurrentHashMap在多线程下的操作

1.一个ConcurrentHashMap由多个segment组成，每一个segment都包含了一个HashEntry数组的hashtable， 每一个segment包含了对自己的HashEntry的操作，比如get，put，replace等操作，这些操作发生的时候，对自己的HashEntry进行锁定。由于每一个segment写操作只锁定自己的HashEntry，所以可能存在多个线程同时写的情况，性能无疑好于只有一个HashEntry锁定的情况。

2.Segment继承了ReentrantLock，所以它就是一种可重入锁（ReentrantLock)。在ConcurrentHashMap，一个Segment就是一个子哈希表，Segment里维护了一个HashEntry数组，并发环境下，对于不同Segment的数据进行操作是不用考虑锁竞争的。

3.在ConcurrentHashMap的remove，put操作还是比较简单的，都是将remove或者put操作交给key所对应的segment去做的，所以当几个操作不在同一个segment的时候就可以并发的进行。在这里面定义了一个volatile count，count = c的操作必须在modCount，table等操作的后面，这样才能保证这些变量操作的可见性。

4.在锁定了这个segement后，get操作并没有使用lock，因为他得到了上面的count，所以count != 0之后，我们可以认为对应的hashtable是最新的，当然由于读取的时候没有加锁，在get的过程中，可能会有更新。当发现根据key去找元素的时候，但发现找得的key对应的value为null，这个时候可能会有其他线程正在对这个元素进行写操作，所以需要在使用锁的情况下在读取一下value，以确保最终的值。
