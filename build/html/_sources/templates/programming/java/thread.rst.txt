ThreadPoolExecutor 
=======================================
构造函数参数
---------------------------------------
1. corePoolSize 核心线程数大小，当线程数 < corePoolSize ，会创建线程执行 runnable
2. maximumPoolSize 最大线程数， 当线程数 >= corePoolSize的时候，会把 runnable 放入 workQueue中
3. keepAliveTime 保持存活时间，当线程数大于corePoolSize的空闲线程能保持的最大时间。
4. unit 时间单位
5. workQueue 保存任务的阻塞队列
6. threadFactory 创建线程的工厂
7. handler 拒绝策略

任务执行顺序
---------------------------------------
1. 当线程数小于 corePoolSize时，创建线程执行任务。
2. 当线程数大于等于 corePoolSize并且 workQueue 没有满时，放入workQueue中
3. 线程数大于等于 corePoolSize并且当 workQueue 满时，新任务新建线程运行，线程总数要小于 maximumPoolSize
4. 当线程总数等于 maximumPoolSize 并且 workQueue 满了的时候执行 handler 的 rejectedExecution。也就是拒绝策略。

阻塞队列
---------------------------------------


+---------------+-----------+-------------+----------+--------------------+
| 方法处理方式  | 抛出异常  | 返回特殊值  | 一直阻塞 | 超时退出           |
+===============+===========+=============+==========+====================+
| 插入方法      | add(e)	| offer(e)    | put(e)	 | offer(e,time,unit) |
+---------------+-----------+-------------+----------+--------------------+
| 移除方法      | remove()  | poll()      | take()	 | poll(time,unit)    |
+---------------+-----------+-------------+----------+--------------------+
| 检查方法      | element() | peek()      | 不可用   | 不可用             |
+---------------+-----------+-------------+----------+--------------------+

1. ArrayBlockingQueue 数组结构组成的有界阻塞队列。
此队列按照先进先出（FIFO）的原则对元素进行排序，但是默认情况下不保证线程公平的访问队列，即如果队列满了，那么被阻塞在外面的线程对队列访问的顺序是不能保证线程公平（即先阻塞，先插入）的。
2. LinkedBlockingQueue一个由链表结构组成的有界阻塞队列
此队列按照先出先进的原则对元素进行排序
3. PriorityBlockingQueue支持优先级的无界阻塞队列
4. DelayQueue支持延时获取元素的无界阻塞队列，即可以指定多久才能从队列中获取当前元素
5. SynchronousQueue不存储元素的阻塞队列，每一个put必须等待一个take操作，否则不能继续添加元素。并且他支持公平访问队列。
6. LinkedTransferQueue由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，多了tryTransfer和transfer方法
transfer方法：
如果当前有消费者正在等待接收元素（take或者待时间限制的poll方法），transfer可以把生产者传入的元素立刻传给消费者。如果没有消费者等待接收元素，则将元素放在队列的tail节点，并等到该元素被消费者消费了才返回。
tryTransfer方法：
用来试探生产者传入的元素能否直接传给消费者。，如果没有消费者在等待，则返回false。和上述方法的区别是该方法无论消费者是否接收，方法立即返回。而transfer方法是必须等到消费者消费了才返回。
7. LinkedBlockingDeque链表结构的双向阻塞队列，优势在于多线程入队时，减少一半的竞争。

拒绝策略
---------------------------------------
ThreadPoolExecutor默认有四个拒绝策略：
1、ThreadPoolExecutor.AbortPolicy() 直接抛出异常RejectedExecutionException
2、ThreadPoolExecutor.CallerRunsPolicy() 直接调用run方法并且阻塞执行
3、ThreadPoolExecutor.DiscardPolicy() 直接丢弃后来的任务
4、ThreadPoolExecutor.DiscardOldestPolicy() 丢弃在队列中队首的任务
当然可以自己继承RejectedExecutionHandler来写拒绝策略.

.. code:: java

	ThreadPoolExecutor threadPool = new ThreadPoolExecutor(5, 6, 3,
        TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(3)
    );
	threadPool.execute(new SelfRunnable("name"));
		
IO线程与业务线程分离
---------------------------------------
* 职责明确化。IO线程只负责IO的读写、编解码和心跳等简单功能，以便及时高效处理IO事件；业务线程负责复杂的业务处理；
* 降低客户端间的相互影响。对于同一IO线程，避免了由于执行客户端A的请求阻塞而无法接受客户端B的请求等现象；
* IO线程负责监听，解码，编码，心跳；IO线程中尽量不要进行业务解析，避免阻塞IO线程；
* 业务线程负责进行业务无处理，中间使用缓冲队列进行解耦；
