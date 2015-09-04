


============================================
序列化与反序列化
============================================

Java序列化与反序列化
============================================

几个需要注意的问题
--------------------------------------------

序列化ID问题:

JVM是否允许反序列化，不仅取决于类路径和功能是否一致，一个非常重要的一点是两个序列化ID是否一致。
如果类完全相同，但是ID不同，依然不能互相序列化与反序列化。

案例：在C/S架构中，使用Facade模式，Client端的Facade Object是利用Server端生成并序列化后通过网络传到Client端。
当服务端Facade进行升级后，将serialVersionUID进行修改，将使得客户端重新获取新的Facade Object。

静态变量是不被序列化的，静态变量是属于类的状态，而序列化是针对Java对象的。

Java在序列化对象进行存储时，如果将同一个对象序列化两次，为了节省空间，第二次写入时只是写对第一次的引用。

对于final变量，因为只能赋值一次，反序列化的时候会重新计算其值，如果反序列化类中将final赋成”古尔丹“，那么在反序列化时就会重新计算成”古尔丹“。但下面这个例子特殊：

.. code:: java
    
    //服务端序列化类
    public class Person implements Serializable{
        private static final long serialVersionUUID = 9187654L;
        public final String name;
        pulic Person(){
            name="地域咆哮";
        }
    }
    //客户端反序列化类
    public class Person implements Serializable{
        private static final long serialVersionUUID = 9187654L;
        public final String name;
        public Person(){
            name = "古尔丹";
        }
    }

反序列化执行后，name的属性值仍然是”地域咆哮“！
这是因为JVM从数据流中获取一个Object对象，然后读取其中的类表述信息，看到final变量，需要重新计算，
于是引用Person类的name值，但是发现竟然反序列化中对name并没有赋值，JVM又”聪明“的引用了旧的name值；
归根到底，是因为反序列化不会执行构造函数，和new有很大区别。


参考资料
============================================
https://www.ibm.com/developerworks/cn/java/j-lo-serial/
