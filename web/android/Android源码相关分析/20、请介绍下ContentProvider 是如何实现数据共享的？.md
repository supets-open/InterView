[使用ContentProvider共享数据](https://blog.csdn.net/u012669600/article/details/41359371)

[ContentProvider是如何实现数据共享的](https://blog.csdn.net/maizangxiangwang/article/details/50738195)

[ContentProvider实现数据共享](https://blog.csdn.net/u010217620/article/details/50592000)

Android提供了ContentProvider，一个程序可以通过实现一个ContentProvider的抽象接口将自己的数据完全暴露出去，而且ContentProviders是以类似数据库中表的方式将数据暴露，也就是说ContentProvider就像一个“数据库”。那么外界获取其提供的数据，也就应该与从数据库中获取数据的操作基本一样，只不过是采用URI来表示外界需要访问的“数据库”。外部访问通过ContentResolver去访问并操作这些被暴露的数据


一个程序可以通过实现一个Content provider的抽象接口将自己的数据完全暴露出去，而且Content provider是以类似数据库中的表的方式将自己的数据暴露。Content provider存储和检索数据，通过它可以让所有的应用程序访问到，这也是应用程序之间唯一共享数据的方法。

要想使应用程序的数据公开化，可通过2种方法：创建一个数据自己的Content Provider或者将你的数据添加到一个已经存在的Content Provider中，前提是有相同数据类型并且有写入Content Provider的权限，Android提供了Content Resolverr，外界的程序可以通过Content Resolver接口访问Content Provider提供的数据。



