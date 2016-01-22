


==========================================================
日志及异常规范定义
==========================================================

日志
==========================================================

日志的功能
----------------------------------------------------------
1. 过程监控－－通过分析程序的执行过程，用于验证程序的执行是否按照既定的方式运行；
2. 问题定位－－通过查看程序错误日志的详细信息以及产生位置，迅速定位问题产生的原因；
3. 业务指导－－通过统计和分析相关的业务、用户行为日志，对业务进行预测指导；
4. 数据恢复－－通过反向执行过程日志，可以将数据可以会滚到之前的状态；
5. 健康检查以及系统优化－－通过查看系统日志，确定程序运行的健康状况，调节系统参数，优化程序性能。

日志级别
----------------------------------------------------------
1. 致命（Fatal）：严重的错误，系统无法正常运行，例如磁盘空间已满；
2. 错误（Error）：系统可以继续运行，但最好要尽快修复的错误，例如Java异常；
3. 警告（Warn）：系统可以正常运行，但需要引起注意的警告信息，例如参数不合法；
4. 信息（Info）：系统运行的主要关键时点的操作信息，一般用于记录业务日志，给管理员或者运营人员看；
5. 调试（Debug）：系统运行中的调试信息，便于开发人员进行错误分析和修正，一般用于程序日志，关心程序操作（细粒度），
   不太关心业务操作（粗粒度）；
6. 跟踪（Trace）：更详细的跟踪日志，当你想了解程序详细流程时使用。

日志文件规范
----------------------------------------------------------
日志文件名必须满足<系统名称>.log.yyyyMMddHH

日志内容规范
----------------------------------------------------------
1. 日志内容必须简洁清晰，是可读性强、干净而且描述性强的，好的日志文件直接表示程序的运行流程；
2. 不能使用Java自有的控制台输出；
3. 建议用StringBuffer进行日志字符串的拼接，但如果需要打印一个类的详细信息，最好写一个toString()或者toPrint()方法，
   避免多次调用get方；
4. 日志不能影响程序的正确运行，不能有副作用；

.. code:: java

    //你确定request不能为null吗？
    log.debug("Processing request with id: {}", request.getId());

5. 不记录集合对象；

.. code:: java

    //因为日志是同步的，可能会由于内存溢出，线程饥饿，lazy initialization，日志文件溢出等造成问题。
    log.debug("Returning users: {}", users);
    //使用如下方式记录ID即可
    log.debug("Returning user ids: {}", collect(users, "id"));
    public static Collection collect(Collection collection, String propertyName) {
        return CollectionUtils.collect(collection, new BeanToPropertyValueTransformer(propertyName));
    }

6. 必须包含上下文信息，具体的ID，名称等，也不要记录只有自己能看懂的 'magic-log' （我就比较喜欢加上#knight标记）；
7. 日志中不能打印任何密码或这私人信息；
8. 使用固定的格式进行输出日志，建议不使用类名，方法名和行数，会对性能有较大影响。例如：%d{HH:mm:ss.SSS} %-5level 
   [%thread][%logger{0}] %m%n；
9. 建议debug或者trace中记录方法参数及返回值，可以帮助调试，并发现线程饥饿，运行效率等。

最佳实践
----------------------------------------------------------

Use Guarded Logging
```````````````````````````````````````````````````````````
"Guarded logging" is a pattern that checks to see if a log statement will result in output before it is executed.
使用大量字符串或者其他对象会降低性能，并倒是GC工作更加频繁。

.. code:: java

    if(LOG.isDebugEnabled()){
        LOG.debug("This is " + "a best practice " + "example");
    }
    //使用SLF4J可以使用如下代码
    log.debug("Found {} records matching filter: '{}'", records, filter);



参考资料
----------------------------------------------------------
https://wiki.base22.com/display/btg/Java+Logging+Standards+and+Guidelines

异常
==========================================================

定义
----------------------------------------------------------
An exception is an event, which occurs during the execution of a program, that disrupts the normal flow of the program's instructions.

异常是程序运行中发生的与正常程序流不同的某个事件。

所有的异常都有一个共同的先祖Throwable, Throwable有两个重要的子类：Exception（异常）和 Error（错误）。

Exception（异常）是应用程序中可能的可预测、可恢复问题。一般大多数异常表示中度到轻度的问题。异常一般是在特定环境下产生的，通常出现在代码
的特定方法和操作中。

Error（错误）表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，当
JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。

Exception 类有一个重要的子类 RuntimeException。RuntimeException 类及其子类表示“JVM 常用操作”引发的错误。

分类
----------------------------------------------------------
Java异常分为受检查异常（Checked）和运行时异常（Unchecked）两种类型。

Checked Exception继承自java.lang.Exception，为受检查异常，表示了关于一个操作的一些有用的信息，这项操作调用者无法控制，但是调用者又需要
知道这些信息。比如，文件系统可能已满，或者远端关闭了连接，又或者访问授权不足以执行这项操作。

Unchecked Exception继承自java.lang.RuntimeException，为运行时异常，通常不显示捕获，例如数组访问越界，空指针异常。

Java异常处理的原则
----------------------------------------------------------

1. 尽可能处理异常，如果条件不允许，则声明向上抛出，一味的声明异常是一种错误和依赖的实践；
2. 具体问题具体解决，异常的部分优点在于能为不同类型的问题提供不同的处理操作，特定异常需要特定的处理模块；不要使用覆盖式的异常处理模块，
   因为没有哪个方法四海皆能用，当代码变化时，完全被忽略了；

.. code:: java

    //错误示例
    try{
        ...
    } catch(Exception e){
        ...
    }

3. 记录可能影响程序的异常，但很有必要，因为可以用来跟踪程序的运行；
4. 根据情形将异常转化为业务上下文，将异常传到不同上下文，需要转换为对新上下文有意义的形式；
5. 不能忽略异常，这是无用工，后续出现问题非常难以排查，导致意向不到的问题；

.. code:: java

    //错误示例
    try{
        Class.forName("business.domain.Customer");
    } catch(ClassNotFoudException e){
    }

6. 有些异常不必捕获和处理，可以通过程序来解决相应的问题；
7. 不要将异常包含在循环块内，会非常占用系统资源；
8. 不要多层次封装后抛出一个非检测异常；
9. 不要多次打印同一个异常，会影响异常的定位；
10. 异常信息应带有上下文，以便快速定位问题；


Checked or Unchecked Exception?
----------------------------------------------------------
一般认为正统的区分方法是：使用Checked Exception表示调用者无法控制的异常信息，而使用Unchecked Exception表示编程上的错误。

Checked exceptions比许多老的语言中使用的错误返回值要好得多。程序员迟早将不再检查错误返回值, 能使用编译器强制正确的错误处理真是太好了。
这样的checked exceptions应该象参数和返回值那样是API的一部分。然而，我不建议使用checked exceptions, 除非调用者能处理它们。Checked 
exceptions尤其不应用来标示发生了一些致命错误, 这些错误不应由调用者来处理。如果调用者代码能做一些有意义的事情, 使用checked exceptions。
如果异常是致命的，或者调用者不能通过捕获异常而有所收获, 使用unchecked exceptions。记住你可以依赖于一个J2EE容器来捕获unchecked exceptions
并记录它们。

未捕获的运行时异常将杀死执行的线程，有时这成为避免使用运行时异常的一个理由。在一些情况下这是一个合理的论据, 但是J2EE应用中这通常不是一
个问题，因为我们极少控制线程，而是让应用服务器来做这个。应用服务器将捕获和处理那些在应用代码中没有捕获的运行时异常，而不会让这些异常挂
起整个Java虚拟机。

Oracle官网上这样说：

.. code::

    Runtime exceptions represent problems that are the result of a programming problem, and as such, the API client code cannot reasonably 
    be expected to recover from them or to handle them in any way. Such problems include arithmetic exceptions, such as dividing by zero; 
    pointer exceptions, such as trying to access an object through a null reference; and indexing exceptions, such as attempting to access 
    an array element through an index that is too large or too small.

    Here's the bottom line guideline: If a client can reasonably be expected to recover from an exception, make it a checked exception. If 
    a client cannot do anything to recover from the exception, make it an unchecked exception.

实践
-----------------------------------------------------------
自定义类型异常时，需要考虑如下几个问题：

1. 你是否真的需要一个现有JDK中没有的异常类型？
2. 调用者是否能从你的异常类中获得有用的信息，并区分其他的异常类？
3. 你的代码是否抛出关联的异常类型？
4. 如果你使用其他人的异常类，调用这有权限获取这个类吗？你的包为什么不是独立而且自包含的？

参考资料
-----------------------------------------------------------
https://docs.oracle.com/javase/tutorial/essential/exceptions/index.html
