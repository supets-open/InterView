
## 如何将一个Java对象序列化到文件里？

### 参考答案

[序列化Serializable和Parcelable的理解和区别](https://blog.csdn.net/zhencheng20082009/article/details/62883739)

使用Java序列化把对象存储到文件中去，再从文件中读取出来；

此时，我们使用ObjectOutputStream和ObjectInputStream来进行对象的读取；

使用ObjectOutputStream对象的writeObject()方法来进行对象的写入；

使用ObjectInputStream对象的readObject()方法来读取对象。