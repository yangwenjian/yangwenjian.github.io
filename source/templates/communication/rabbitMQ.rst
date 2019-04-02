


=======================================
RabbitMQ
=======================================
* Broker：简单来说就是消息队列服务器实体。
* Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
* Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
* Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
* Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
* Producer：消息生产者，就是投递消息的程序。
* Consumer：消息消费者，就是接受消息的程序。
* Channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。


Exchange
=======================================

* 客户端连接到消息队列服务器，打开一个channel。
* 客户端声明一个exchange，并设置相关属性。
* 客户端声明一个queue，并设置相关属性。
* 客户端使用routing key，在exchange和queue之间建立好绑定关系。
* 客户端投递消息到exchange。
* exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。

Direct Exchange
---------------------------------------
 
.. image:: ./images/rabbitmq_direct_exchange.png

这种类型的Exchange，通常是将同一个message以一种循环的方式分发到不同的Queue，即不同的消费者手中，使用这种方式，值得注意的是message在消费者之间做了一个均衡，而不是说message在Queues之间做了均衡

Fanout Exchange
---------------------------------------

.. image:: ./images/rabbitmq_fanout_exchange.png

使用这种类型的Exchange，会忽略routing key的存在，直接将message广播到所有的Queue中。
第一：大型玩家在玩在线游戏的时候，可以用它来广播重大消息。这让我想到电影微微一笑很倾城中，有款游戏需要在世界上公布玩家重大消息，也许这个就是用的MQ实现的。这让我不禁佩服肖奈，人家在大学的时候就知道RabbitMQ的这种特性了。
第二：体育新闻实时更新到手机客户端。
第三：群聊功能，广播消息给当前群聊中的所有人。

Topic Exchange
---------------------------------------
Topic Exchange是根据routing key和Exchange的类型将message发送到一个或者多个Queue中，我们经常拿他来实现各种publish/subscribe，即发布订阅，这也是我们经常使用到的ExchangeType。

首先建立queue和exchange，然后将两者按照routingkey进行绑定，之后按自定义的规则发送到该exchange，绑定时符合的queue将会收到该信息。
另外rabbitMQ对于同一队列的多个消费者，采用轮循的机制进行分发消息。

新闻的分类更新
同意任务多个工作者协调完成
同一问题需要特定人员知晓
Topic Exchange的使用场景很多，我们公司就在使用这种模式，将足球事件信息发布，需要使用这些事件消息的人只需要绑定对应的Exchange就可以获取最新消息。

Headers Exchange
---------------------------------------
Headers Exchange不同于上面三种Exchange，它是根据Message的一些头部信息来分发过滤Message，忽略routing key的属性，如果Header信息和message消息的头信息相匹配，那么这条消息就匹配上了。
 
最佳实践
=======================================

.. code:: java
	
	@Component
	@RabbitListener(queues = "topic.messages")
	public class TopicMessagesReceiver {
		// 接受者
		@RabbitHandler
		public void process(User user) {
			System.out.println("TopicMessagesReceiver Consumer 2 : " + user);
		}
	 
	}
	


