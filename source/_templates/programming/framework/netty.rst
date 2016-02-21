


===================================================================
Netty
===================================================================
Netty is an asynchronous event-driven network application framework for rapid development of maintainable high 
performance protocol servers & clients.

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


参考资料
===================================================================
http://netty.io/
