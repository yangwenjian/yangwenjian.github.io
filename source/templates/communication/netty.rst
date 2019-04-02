


===================================================================
Netty
===================================================================
Netty is an asynchronous event-driven network application framework for rapid development of maintainable high 
performance protocol servers & clients.

Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. 

NIO
---------------------------------------
与 Socket 类和 ServerSocket 类相对应，NIO 也提供了 SocketChannel 和
ServerSocketChannel 两种不同的套接字通道实现；

//.. image:: images/netty.png

1. 构造ServerBootstrap；
2. EventLoop的职责是处理所有注册到本线程多路复用器Selector上的Channel；EventLoop的职责不仅仅是处理网络I/O事件，用户自定义的Task和定时任务Task也统一由EventLoop负责处理，这样线程模型就实现了统一。也不会启其他异步线程，避免多线程并发操作和锁竞争；
3. 通过反射创建NioServerSocketChannel；
4. ChannelPipline本质是一个负责处理网络事件的职责链，负责管理和执行Channelhandler，网络事件以事件流的形式在ChannelPipeline中流转，根据执行策略调度Channelhandler的执行。
5. 添加并设置ChannelHandler，完成消息编码、心跳、安全认证、TSL/SSL认证、流量控制和流量整形等。
6. 在绑定监听端口之前系统会做一系列的初始化和检测工作，之后启动端口，将ServerSocketChannel注册到Selector上监听客户端连接；
7. Selector轮询，由Reactor线程NioEventLoop负责调度和执行Selector轮询操作。
8. 轮询到准备就绪的Channel，就由Reactor线程NioEventLoop执行ChannelPipline的相应方法；
9.执行Netty系统ChannelHandler和用户定制ChannelHandler；

Please keep in mind that it is the handler's responsibility to release any reference-counted object passed to the handler.

.. code::
    
    ReferenceCountUtil.release(msg);


超时设置
---------------------------------------
* 使用了 IdleStateHandler ，分别设置了读、写超时的时间
* 定义了一个 HeartbeatServerHandler 处理器，用来处理超时时，发送心跳

.. code:: java

    public class HeartbeatHandlerInitializer extends ChannelInitializer<Channel> {
 
        private static final int READ_IDEL_TIME_OUT = 4; // 读超时
        private static final int WRITE_IDEL_TIME_OUT = 5;// 写超时
        private static final int ALL_IDEL_TIME_OUT = 7; // 所有超时
     
        @Override
        protected void initChannel(Channel ch) throws Exception {
            ChannelPipeline pipeline = ch.pipeline();
            pipeline.addLast(new IdleStateHandler(READ_IDEL_TIME_OUT,
                    WRITE_IDEL_TIME_OUT, ALL_IDEL_TIME_OUT, TimeUnit.SECONDS)); // 1
            pipeline.addLast(new HeartbeatServerHandler()); // 2
        }
    }
    
* 定义了心跳时，要发送的内容
* 判断是否是 IdleStateEvent 事件，是则处理
* 将心跳内容发送给客户端

.. code:: java

    public class HeartbeatServerHandler extends ChannelInboundHandlerAdapter {
 
        // Return a unreleasable view on the given ByteBuf
        // which will just ignore release and retain calls.
        private static final ByteBuf HEARTBEAT_SEQUENCE = Unpooled
                .unreleasableBuffer(Unpooled.copiedBuffer("Heartbeat",
                        CharsetUtil.UTF_8));  // 1
     
        @Override
        public void userEventTriggered(ChannelHandlerContext ctx, Object evt)
                throws Exception {
     
            if (evt instanceof IdleStateEvent) {  // 2
                IdleStateEvent event = (IdleStateEvent) evt;  
                String type = "";
                if (event.state() == IdleState.READER_IDLE) {
                    type = "read idle";
                } else if (event.state() == IdleState.WRITER_IDLE) {
                    type = "write idle";
                } else if (event.state() == IdleState.ALL_IDLE) {
                    type = "all idle";
                }
     
                ctx.writeAndFlush(HEARTBEAT_SEQUENCE.duplicate()).addListener(
                        ChannelFutureListener.CLOSE_ON_FAILURE);  // 3
     
                System.out.println( ctx.channel().remoteAddress()+"超时类型：" + type);
            } else {
                super.userEventTriggered(ctx, evt);
            }
        }
    }
    
海量数据推送要点
=======================================
1. 设置linux单进程句柄数（ulimit -a）
2. 当心CLOSE_WAIT

.. code::

    客户端的重连间隔需要合理设置，防止连接过于频繁导致的连接失败（例如端口还没有被释放）；  
    客户端重复登陆拒绝机制；
    服务端正确处理 I/O 异常和解码异常等，防止句柄泄露。

close_wait 是被动关闭连接是形成的，根据 TCP 状态机，服务器端收到客户端发送的 FIN，TCP 协议栈会自动发送 ACK，链接进入 close_wait 状态。但如果服务器端不执行 socket 的 close() 操作，状态就不能由 close_wait 迁移到 last_ack，则系统中会存在很多 close_wait 状态的连接。通常来说，一个 close_wait 会维持至少 2 个小时的时间（系统默认超时时间的是 7200 秒，也就是 2 小时）。如果服务端程序因某个原因导致系统造成一堆 close_wait 消耗资源，那么通常是等不到释放那一刻，系统就已崩溃

.. code::

    设计要点 1：不要在 Netty 的 I/O 线程上处理业务（心跳发送和检测除外）。Why? 对于 Java 进程，线程不能无限增长，这就意味着 Netty 的 Reactor 线程数必须收敛，非阻塞I/O，线程数经验值是 [CPU 核数 + 1，CPU 核数\*2 ] 之间
    
.. code:: 

    设计要点 2：在 I/O 线程上执行自定义 Task 要当心。Netty 的 I/O 处理线程 NioEventLoop 支持Runnable和ScheduleFutureTask，通过 setIoRatio(int ioRatio)方法设置占用CPU比例
    
.. code::

    IdleStateHandler 使用要当心，在心跳的业务逻辑处理中，无论是正常还是异常场景，处理时延要可控，防止时延不可控导致的 NioEventLoop 被意外阻塞
    
合理的心跳周期
```````````````````````````````````````
过长会导致链接释放，造成频繁的客户端重连，但是也不能设置太短，否则在当前缺乏统一心跳框架的机制下很容易导致信令风暴，建议180s，微信使用300s；

合理设置接收和发送缓冲区容量
```````````````````````````````````````
幸运的是，Netty 提供的 ByteBuf 支持容量动态调整，对于接收缓冲区的内存分配器，Netty 提供了两种：

FixedRecvByteBufAllocator：固定长度的接收缓冲区分配器，由它分配的 ByteBuf 长度都是固定大小的，并不会根据实际数据报的大小动态收缩。但是，如果容量不足，支持动态扩展。动态扩展是 Netty ByteBuf 的一项基本功能，与 ByteBuf 分配器的实现没有关系；
AdaptiveRecvByteBufAllocator：容量动态调整的接收缓冲区分配器，它会根据之前 Channel 接收到的数据报大小进行计算，如果连续填充满接收缓冲区的可写空间，则动态扩展容量。如果连续 2 次接收到的数据报都小于指定值，则收缩当前的容量，以节约内存。

内存池
```````````````````````````````````````
Netty4 的 PooledByteBufAllocator 进行 GC 优化
使用内存池之后，内存的申请和释放必须成对出现，即 retain() 和 release() 要成对出现，否则会导致内存泄露。
值得注意的是，如果使用内存池，完成 ByteBuf 的解码工作之后必须显式的调用 ReferenceCountUtil.release(msg) 对接收缓冲区 ByteBuf 进行内存释放，否则它会被认为仍然在使用中，这样会导致内存泄露。

当心“日志隐形杀手”
```````````````````````````````````````
以最常用的 log4j 为例，尽管它支持异步写日志（AsyncAppender），但是当日志队列满之后，它会同步阻塞业务线程，直到日志队列有空闲位置可用

.. code::

    synchronized (this.buffer){
        while (true) {
            int previousSize = this.buffer.size();
            if (previousSize < this.bufferSize) {
                this.buffer.add(event);
                if (previousSize != 0) break;
                this.buffer.notifyAll();
                break;
            }
            boolean discard = true;
            if ((this.blocking) && (!Thread.interrupted()) && (Thread.currentThread() != this.dispatcher)) // 判断是业务线程 
            {
                try {
                    this.buffer.wait();// 阻塞业务线程 
                    discard = false;
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

            }
        }
    }

TCP 参数优化
```````````````````````````````````````
常用的 TCP 参数，例如 TCP 层面的接收和发送缓冲区大小设置，在 Netty 中分别对应 ChannelOption 的 SO_SNDBUF 和 SO_RCVBUF，需要根据推送消息的大小，合理设置，对于海量长连接，通常 32K 是个不错的选择。

另外一个比较常用的优化手段就是软中断，如图所示：如果所有的软中断都运行在 CPU0 相应网卡的硬件中断上，那么始终都是 cpu0 在处理软中断，而此时其它 CPU 资源就被浪费了

大于等于 2.6.35 版本的 Linux kernel 内核，开启 RPS，网络通信性能提升 20% 之上。RPS 的基本原理：根据数据包的源地址，目的地址以及目的和源端口，计算出一个 hash 值，然后根据这个 hash 值来选择软中断运行的 cpu。从上层来看，也就是说将每个连接和 cpu 绑定，并通过这个 hash 值，来均衡软中断运行在多个 cpu 上，从而提升通信性能。

JVM参数
```````````````````````````````````````
-Xmx:JVM 最大内存需要根据内存模型进行计算并得出相对合理的值；
GC 相关的参数: 例如新生代和老生代、永久代的比例，GC 的策略，新生代各区的比例等，需要根据具体的场景进行设置和测试，并不断的优化，尽量将 Full GC 的频率降到最低。

Jmeter压测及调试
=======================================

最佳实践
=======================================
当我们在业务线程里通过 ChannelHandlerContext.write 发送消息的时候，Netty 4 在将消息发送事件调度到 ChannelPipeline 的时候，首先将待发送的消息封装成一个 Task，然后放到 NioEventLoop 的任务队列中，由 NioEventLoop 线程异步执行。后续所有 handler 的调度和执行，包括消息的发送、I/O 事件的通知，都由 NioEventLoop 线程负责处理。

ByteBuf 在业务线程中申请，在后续的 ChannelHandler 中释放，ChannelHandler 是由 Netty 的 I/O 线程 (EventLoop) 执行的，因此内存的申请和释放不在同一个线程中，导致内存泄漏

申请之后一定要记得释放，Netty 自身 Socket 读取和发送的 ByteBuf 系统会自动释放，用户不需要做二次释放；如果用户使用 Netty 的内存池在应用中做 ByteBuf 的对象池使用，则需要自己主动释放；
避免错误的释放：跨线程释放、重复释放等都是非法操作，要避免。特别是跨线程申请和释放，往往具有隐蔽性，问题定位难度较大；
防止隐式的申请和分配：之前曾经发生过一个案例，为了解决内存池跨线程申请和释放问题，有用户对内存池做了二次包装，以实现多线程操作时，内存始终由包装的管理线程申请和释放，这样可以屏蔽用户业务线程模型和访问方式的差异。谁知运行一段时间之后再次发生了内存泄露，最后发现原来调用 ByteBuf 的 write 操作时，如果内存容量不足，会自动进行容量扩展。扩展操作由业务线程执行，这就绕过了内存池管理线程，发生了“引用逃逸”；
避免跨线程申请和使用内存池，由于存在“引用逃逸”等隐式的内存创建，实际上跨线程申请和使用内存池是非常危险的行为。尽管从技术角度看可以实现一个跨线程协调的内存池机制，甚至重写 PooledByteBufAllocator，但是这无疑会增加很多复杂性，通常也使用不到。如果确实存在跨线程的 ByteBuf 传递，而且无法保证 ByteBuf 在另一个线程中会重新分配大小等操作，最简单保险的方式就是在线程切换点做一次 ByteBuf 的拷贝，但这会造成性能下降。
比较好的一种方案就是如果存在跨线程的 ByteBuf 传递，对 ByteBuf 的写操作要在分配线程完成，另一个线程只能做读操作。操作完成之后发送一个事件通知分配线程，由分配线程执行内存释放操作

Netty升级问题
---------------------------------------
在 Netty 3 中，消息发送性能统计相关 Handler，编码 Handler方法都是由业务线程负责执行；而在 Netty 4 中，则是由 NioEventLoop(I/O) 线程执行。对于某个链路，业务是拥有多个线程的线程池，而 NioEventLoop 只有一个，所以执行效率更低，返回给客户端的应答时延就大。时延增大之后，自然导致系统并发量降低，性能下降

线程安全问题
---------------------------------------
ChannelHandler's 的方法不会被 Netty 并发调用；
用户不再需要对 ChannelHandler 的各个方法做同步保护；
ChannelHandler 实例不允许被多次添加到 ChannelPiple 中，否则线程安全将得不到保证

如果 ChannelHandler 被注解为 @Sharable，全局只有一个 handler 实例，它会被多个 Channel 的 Pipeline 共享，会被多线程并发调用，因此它不是线程安全的；
如果存在跨 ChannelHandler 的实例级变量共享，需要特别注意，它可能不是线程安全的

SocketIO
===================================================================
SocketIO是用同步的方式等待客户端连接，当有客户端连接时，分配一个Handler进行处理。

.. code:: java

    while(true){
        Socket socket = serverSocket.accept();
        // Pass the socket to the RequestHandler thread for processing
        RequestHandler requestHandler = new RequestHandler(socket);
        requestHandler.start();
    }

    class RequestHandler extends Thread{
        ...
        public void run(){
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream());
            out.println("Echo Server 1.0");
            out.flush();
            String line = in.readLine();
            while (line != null && line.length() > 0) {
                out.println("Server Echo: " + line);
                out.flush();
               line = in.readLine();
            }
            in.close();
            out.close();
            socket.close();
        }
    }



NIO
===================================================================
NIO的本质是Socket连接，但使用了JDK7提供的异步处理框架FutureTask，调用后立即返回，避免阻塞。

.. code:: java

    final AsynchronousServerSocketChannel listener = AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(8017));
    listener.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
        public void completed(AsynchronousSocketChannel ch, Void att) {
            //accept the next connection
            listener.accept(null, this);
            ByteBuffer byteBuffer = ByteBuffer.allocate(4096);
            try{
                int bytesRead = ch.read(byteBuffer).get(20, TimeUnit.SECONDS);
                boolean running = true;
                while (bytesRead != -1 && running) {
                    if (byteBuffer.position() > 2) {
                        byteBuffer.flip();
                        byte[] lineBytes = new byte[bytesRead];
                        byteBuffer.get(lineBytes, 0, bytesRead);
                        String line = new String(lineBytes);
                        ch.write(ByteBuffer.wrap(line.getBytes()));
                        byteBuffer.clear();
                        bytesRead = ch.read(byteBuffer).get(20, TimeUnit.SECONDS);
                    }else{
                        running = false;
                    }
                }
            }catch(Exception e){
                ...//handle exception
            }
            ...//close ch
        }
    });


channelFuture
---------------------------------------
在Netty中所有的IO操作都是异步的。这意味着所有的IO调用将立即返回，但不保证在调用结束时请求的IO操作都已经执行完毕。而是在请求操作处理完成、失败或者取消时返回一个ChannelFuture来通知。


最佳实践
===================================================================
1. 使用netty作为web前端推送消息框架。
   在云医院的web项目中，因为前台定时刷新数据库带来的性能问题比较严重，因此选择使用推送方式发送消息。
   （其实我觉得是设计的问题，因为前台完全可以读redis而不去读取数据库内容，这样就可以提高效率）

使用netty框架也能解决问题，利用前台的JS客户端进行连接，有消息的话进行推送。

.. code::

    socket = io.connect('http://10.32.170.68:9082');
    socket.on('receive', function(data){
        var type = $.parseJSON(data).type;
        ...
    })

这里加入的认证机制，通过用户第一次登录时需要进行身份认证，认证后，建立session与client的映射关系，保存到缓存中。

.. code::java

    public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg)
            throws Exception {
        System.out.println("server channelRead..");
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        System.out.println("The time server receive order:" + body);
        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new Date(
                System.currentTimeMillis()).toString() : "BAD ORDER";
        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        ctx.write(resp);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        System.out.println("server channelReadComplete..");
        ctx.flush();//刷新后才将数据发出到SocketChannel
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)
            throws Exception {
        System.out.println("server exceptionCaught..");
        ctx.close();
    }

}

GenericFutureListener
---------------------------------------

.. code:: java

    public synchronized ListenableFuture<Void> connect ()
    {
        if ( this.connectFuture != null )
        {
            return this.connectFuture;
        }

        final ChannelFuture channelFuture = this.bootstrap.connect ( this.address );
        this.connectFuture = SettableFuture.create ();

        channelFuture.addListener ( new GenericFutureListener<ChannelFuture> () {

            @Override
            public void operationComplete ( final ChannelFuture future ) throws Exception
            {
                handleOperationComplete ( Client.this.connectFuture, future );
            }
        } );

        return this.connectFuture;
    }
    
.. code:: java

    protected void setupWithTest() {
      ChannelFuture future = boot.connect(uri.host(), port);
      future.addListener(
          new GenericFutureListener<ChannelFuture>() {

            public void operationComplete(ChannelFuture f) {
              if (f.isSuccess()) {
                channel = (HTTPChannel) f.channel();
                testConnection();
                onTestBell.promise(onConnectBell);
              } else {
                onConnectBell.ring(f.cause());
              }
            }
          });
    }
    
.. code:: java

    public void kickPlayerFromServer(String reason)
    {
        final ChatComponentText chatcomponenttext = new ChatComponentText(reason);
        this.netManager.sendPacket(new S40PacketDisconnect(chatcomponenttext), new GenericFutureListener < Future <? super Void >> ()
        {
            public void operationComplete(Future <? super Void > p_operationComplete_1_) throws Exception
            {
                NetHandlerPlayServer.this.netManager.closeChannel(chatcomponenttext);
            }
        }, new GenericFutureListener[0]);
        this.netManager.disableAutoRead();
        Futures.getUnchecked(this.serverController.addScheduledTask(new Runnable()
        {
            public void run()
            {
                NetHandlerPlayServer.this.netManager.checkDisconnected();
            }
        }));
    }
    
.. code:: java

    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {

                public ChannelPipeline getPipeline() {
                    return Channels.pipeline(new ObjectEncoder(),
                            new SimpleChannelUpstreamHandler(){
                        @Override
                        public void channelDisconnected(ChannelHandlerContext ctx, ChannelStateEvent e) {
                            println("D          bootstrap.connect();
                                }
                            }, NettyClient.RECONNECT_DELAY, TimeUnit.SECONDS);
                        }

                        @Override
                        public void channelConnected(ChannelHandlerContext ctx, ChannelStateEvent e) {
                            if (startTime < 0) {
                                startTime = System.currentTimeMillis();
                            }
                            println("连接到: " + getRemoteAddress());
                            flag=true;
                            channel = e.getChannel();
                        }

                        @Override
                        public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) {
                            Throwable cause = e.getCause();
                            if (cause instanceof ConnectException) {
                                startTime = -1;
                                println("连接失败，原因: " + cause.getMessage());
                            }
                            if (cause instanceof ReadTimeoutException) {
                                //连接是好的，但是超时
                                println("超时...");
                            } else {
                                cause.printStackTrace();
                            }
                            ctx.getChannel().close();
                        }
                    });
                    
                }
            });
    }

参考资料
===================================================================
http://netty.io/
https://waylau.com/netty-websocket-chat/
https://www.infoq.cn/article/netty-million-level-push-service-design-points
https://www.infoq.cn/article/the-multithreading-of-netty-cases
