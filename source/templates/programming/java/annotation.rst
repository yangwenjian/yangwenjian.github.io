



=============================================
Java Annotation
=============================================
Java Annotation is an import part in jdk

反射机制
=============================================
首先介绍下反射机制，是java的一个重要特性，是其他语言没有的。
反射，顾名思义就是利用一些外围信息获取关键性的内部信息。

我所学习的反射，基本用于程序的解藕，考虑如下例子：

有个类型为Person，属性有name, gender, height。
我想获取某个person对象的所有信息，放入一个Map中，必然要知道person的内部实现，如果person发生变动，必然这段代码也要发生变动...

但是如果我用反射的方法，就可以避免这个问题：

::

    Field[] fields = person.getClass().getDeclaredFields();
    for(Field field : fields){
        field.setAccessible(true);
        String name = field.getName();
        Object value = field.get(person);
        map.put(name, value);
    }


知识梳理
=============================================
AOP(Aspect Oriented Programming),面向切面的编程。
我理解为不破坏原有任何程序的情况下，能够自由的控制程序的运行方式，就好比横向切开程序，加入想要执行的代码或者属性等。

APT(Annotaion Processing Tool)注解分析器。

元数据，在文档编制，编译检查，代码分析中起作用。

反射机制，读取注释信息的关键机制。
