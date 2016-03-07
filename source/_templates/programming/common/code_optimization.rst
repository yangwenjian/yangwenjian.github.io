


============================================
Code Optimization
============================================
The biggest speedup you'll ever get with a program will be when you first get it working.

                                                                    --John K. Ousterhout

Loop Unrolling
============================================

基本原则
============================================
1. 优化的第一项原则就是保证代码在所有条件下运行正确，效率高但有bug的代码根本没有用。
2. 必须写出简洁清晰的代码，在review或者modification时可以更有效率。
3. 使用合适的数据结构和算法。
4. 编写出编译器能给出最大优化的代码。
5. 使用任务分割和并行计算来提高效率。

最佳实践
============================================
不同语言的代码优化方式不同，我喜欢的java、c++、c，我们将来讲讲代码优化的方法和效果。

* 减少程序中不必要的代码，包括无意义的调用、条件测试、内存引用等；
* 使程序生成的机器代码可以被处理器并行计算；
* 使用Code Motion将不变的因素移动到循环外部；
* 减少过程调用（Reducing Procedure Calls）；

java code optimazation
--------------------------------------------
1. 尽量重用对象，如连接String的时候尽量使用StringBuffer和StringBuilder，可以减少系统生成对象的时间，还能减少回收这些对象的时间；
2. 尽量使用局部变量，局部变量保存在Stack中，而静态和实例变量存储在Heap中，Heap访问比Stack访问慢，而且局部变量能被编译器优化；
3. 减少对代码的重复计算，有计算的情况保留结果值，比重新计算节省资源;
4. 不要在循环中使用try catch语句块;

云医院web项目中的优化策略
````````````````````````````````````````````
1. 建立单字段索引，联合索引，优化mysql语句；

.. code::

    SELECT 
        tb1.service_instance_code 
    FROM
        ch_ser_service_instance AS tb1,
        ch_ser_service_instance_owner AS tb2 
    WHERE tb1.service_instance_code = tb2.service_instance_code 
    AND tb2.owner_type = 'DOCTOR' 
    AND tb2.owner_code = 'C569DAACBE4B0AAC03A718B52' 
    ORDER BY service_status DESC,
    service_start_date DESC 
    IMIT 0, 8 

    在两个表中100w数据量以上时，执行时间在27s左右，优化为如下代码后，执行时间变为2s左右

    SELECT 
        service_instance_code 
    FROM
        (SELECT 
            tb1.service_instance_code,
            tb1.service_status,
            tb1.service_start_date 
        FROM
            ch_ser_service_instance AS tb1,
            ch_ser_service_instance_owner AS tb2 
        WHERE tb1.service_instance_code = tb2.service_instance_code 
        AND tb2.owner_type = 'DOCTOR' 
        AND tb2.owner_code = 'C569DAACBE4B0AAC03A718B52') tb3
    ORDER BY service_status DESC,
        service_start_date DESC 
    LIMIT 0, 8 

2. mvc中的intercepor里尽量减少代码逻辑和数据库访问，因为每次访问DispatchServlet时都会执行interceptor中的代码，如果里面有过多的
   数据库访问，造成用户数大量并发中出现严重的性能问题，在云医院的interceptor中，就多次访问数据库并且还进行了远程调用phr接口，
   修改后进行一次调用后进行缓存，存储到redis中或者session中都可以解决这个问题。


Java本身在编译器上做了很多优化，而且是一门成熟的语言，在编写代码时不像C++那样需要自己管理内存，简化了很多优化工作，同时给程序员
表现的机会也没那么多，只要注意几个浪费空间和时间点，其余可以在算法上做优化，语言本身的优化可能还要看多线程这块如何进行。







参考资料： 
 http://nakata-yf.iteye.com/blog/23545 
