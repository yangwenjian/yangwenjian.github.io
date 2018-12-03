


===================================================================
Netty
===================================================================
Netty is an asynchronous event-driven network application framework for rapid development of maintainable high 
performance protocol servers & clients.

Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. 

Please keep in mind that it is the handler's responsibility to release any reference-counted object passed to the handler.

.. code::
    
    ReferenceCountUtil.release(msg);

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
