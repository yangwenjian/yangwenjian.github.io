


============================================
Code Optimization
============================================
The biggest speedup you'll ever get with a program will be when you first get it working.

                                                                    --John K. Ousterhout

一、 基本原则
============================================
1. 优化的第一项原则就是保证代码在所有条件下运行正确，效率高但有bug的代码根本没有用。
2. 必须写出简洁清晰的代码，在review或者modification时可以更有效率。
3. 使用合适的数据结构和算法。
4. 编写出编译器能给出最大优化的代码。
5. 使用任务分割和并行计算来提高效率。

二、 优化方案的演变
============================================
我们度量循环程序中的速度，使用cycles per element（“CPE”），意思是每次迭代需要的时钟周期数。
我们考虑将一个vector中的所有元素进行相加或者相乘，使用如下方案，CPE在29左右。

.. code:: cpp

    typedef int data_t;
    typedef struct{
        long int len;
        data_t *data;
    } vec_rec, *vec_prt;

    #define IDENT 0
    #define OP +

    void combine1(vec_ptr v, data *dest){
        long int i;
        *dest = IDENT;
        for(i = 0; i < vec_length(v); i++){
            data_t val;
            get_vec_element(v, i, &val);
            *dest = *dest OP val;
        }
    }

1. Code Motion
--------------------------------------------
使用*Code Motion*技术，然vector的长度在整个计算过程中没有发生任何变化，我们就计算一次vector的长度，将其移出循环。
将CPE提高8～12左右（integer～double）。

.. code:: cpp

    void combine2(vec_prt v, data *dest){
        long int i;
        long int length = vec_length(v);
        *dest = IDENT;
        for(i = 0; i < length; i++){
            data_t val;
            get_vec_element(v, i, &val);
            *dest = *dest OP val;
        }
    }

2. Reducing Procedure Calls
--------------------------------------------
使用*Reducing Procedure Calls*方式，可以减少循环内的方法调用。
可以将加法的CPE降到6左右。

.. code:: cpp
    
    data_t *get_vec_start(vec_prt v){
        return v->data;
    }
    void combine3(vec_prt v, data *dest){
        long int i;
        long int length = vec_length(v);
        data_t *data = get_vec_start(v);
        *dest = IDENT;
        for(i = 0; i < length; i++){
            *dest = *dest OP data[i];
        }
    }

3. Eliminating Unneeded Memory Reference
--------------------------------------------
减少不必要的内存使用，因为汇编代码不能直接计算内存中的数据，需要进行提取到寄存器中，在进行计算和存储。
通过减少内存的引用，可以将CPE提高到2～5(integer~double)。但编译器并不会进行这样的优化，因为编译器是非常小心的。
考虑如下情况，\*dest指向v中的某个数据，这样的话就会改变数组的值导致结果的不一致。

.. code:: cpp

    void combine4(vec_prt v, data *dest){
        long int i;
        long int length = vec_length(v);
        data_t *data = get_vec_start(v);
        data_t acc = IDENT;
        for( i = 0; i < length; i++){
            acc = acc OP data[i];
        }
        *dest = acc;
    }

通过减少内存的引用，可以将CPE提高到2～5(integer~double)。但编译器并不会进行这样的优化，因为编译器是非常小心的。
考虑如下情况，\*dest指向v中的某个数据，这样的话就会改变数组的值导致结果的不一致。

通过加入编译选项-O2,可以将CPE减少到3～5（integer～double），汇编代码如下，减少了从内存读取的步骤：

.. code:: 

    // i in %rdx, data in %rax, limit in %rbp, dest at %rx12, product in %xmm0
    .L560:                                  loop:
        mulss   (%rax, %rdx, 4), %xmm0           Multiply product by data[i]
        addq    $1, %rdx                        Increment i
        cmpq    %rdx, %rbp                      Compare limit:i
        movss   %xmm0, (%r12)                   Store product at dest
        jg      .L560                           If >, goto loop

与-O1的情况进行比较，结果如下：

.. code::

    .L498:                                  loop:
        movss (%rbp), %xmm0                     Read product from dest
        mulss (%rax, &rdx, 4), %xmm0            Multiply product by data[i]
        movss %xmm0, (%rbp)                     Store product at dest
        cmpq %rdx, %r12                         Compare i:limit
        jg   .L498                              if >, goto loop

4. Understanding Modern Processors
--------------------------------------------
We must exploit the microarchitecture of the processor(instruction-level parralelism). Modern processor can employ a 
technique known as branch prediction. Using speculative execution, the processor begins fetching and decoding instructions
at where it predicts the branch will go, and even begins executing these operations. If it later determines that the branch
was predicted incorrectly, it resets the state to that at branch point and begins fetching and executing instructions in 
the other derection.

But the result will not be stored in registers or data memory until the processor can be certain that these instructions 
should actually been executed.

Incorrect prediction(misprediction) incurs a significant cost in performance. It takes a while before the new instructions 
can be fetched, decoded, and sent to the execution units.

The execution units can send results directly to each other, to expedite the communication of reults from one instruction 
to another.





最佳实践
============================================

云医院web项目中的优化策略
--------------------------------------------
1. 尽量重用对象，如连接String的时候尽量使用StringBuffer和StringBuilder，可以减少系统生成对象的时间，还能减少回收这些对象的时间；
2. 尽量使用局部变量，局部变量保存在Stack中，而静态和实例变量存储在Heap中，Heap访问比Stack访问慢，而且局部变量能被编译器优化；
3. 减少对代码的重复计算，有计算的情况保留结果值，比重新计算节省资源;
4. 不要在循环中使用try catch语句块;
5. 建立单字段索引，联合索引，优化mysql语句；

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

6. mvc中的intercepor里尽量减少代码逻辑和数据库访问，因为每次访问DispatchServlet时都会执行interceptor中的代码，如果里面有过多的
   数据库访问，造成用户数大量并发中出现严重的性能问题，在云医院的interceptor中，就多次访问数据库并且还进行了远程调用phr接口，
   修改后进行一次调用后进行缓存，存储到redis中或者session中都可以解决这个问题。


Java本身在编译器上做了很多优化，而且是一门成熟的语言，在编写代码时不像C++那样需要自己管理内存，简化了很多优化工作，同时给程序员
表现的机会也没那么多，只要注意几个浪费空间和时间点，其余可以在算法上做优化，语言本身的优化可能还要看多线程这块如何进行。







参考资料： 
 http://nakata-yf.iteye.com/blog/23545 
