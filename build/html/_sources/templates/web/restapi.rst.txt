


==========================================
Restful
==========================================

REST
==========================================
REST: Representational State Transfor，是当前比较流行技术，可以降低耦合度、复杂性，并且提高兼容性，可移植性。一个功能强、
性能好、适宜通讯的，符合Rest的约束条件和原则的架构，称为Restful架构。

* 客户-服务器（Client-Server）
  通信只能由客户端单方面发起，表现为请求-响应的形式；
* 无状态（Stateless）
  通信的会话状态（Session State）应该全部由客户端负责维护；
* 缓存（Cache）
  响应的内容可以在通信链的某处缓存，以改善网络效率；
* 统一接口（Uniform Interface）
  通信链的组件之间通过统一接口相互通信，以提高交互的可见性；
* 分层系统（Layered System）
  通过限制组件的行为（即，每个组件只能“看到”与其交互的紧邻层），将架构分解为若干等级的层；
* 按需代码（Code-On-Demand, Optional）
  支持通过下载并执行一些代码（例如Java Applet、Flash或JavaScript），对客户端的功能进行扩展。

Rest架构特点
----------------------------------------
1. 简单性
   采用REST架构风格，对于开发、测试、运维人员来说，都会更简单。可以充分利用大量HTTP服务器端和客户端开发库、Web功能测试/性能
   测试工具、HTTP缓存、HTTP代理服务器、防火墙。这些开发库和基础设施早已成为了日常用品，不需要什么火箭科技（例如神奇昂贵的应
   用服务器、中间件）就能解决大多数可伸缩性方面的问题；
2. 可伸缩性
   充分利用好通信链各个位置的HTTP缓存组件，可以带来更好的可伸缩性。其实很多时候，在Web前端做性能优化，产生的效果不亚于仅仅
   在服务器端做性能优化，但是HTTP协议层面的缓存常常被一些资深的架构师完全忽略掉；
3. 松耦合
   统一接口+超文本驱动，带来了最大限度的松耦合。允许服务器端和客户端程序在很大范围内，相对独立地进化。对于设计面向企业内网
   的API来说，松耦合并不是一个很重要的设计关注点。但是对于设计面向互联网的API来说，松耦合变成了一个必选项，不仅在设计时应
   该关注，而且应该放在最优先位置。


RestAPI的请求与返回
========================================
Json定义的随意度比较大，如何解析，要看字符串的具体格式和解析方式。
如何解析返回的json串，这里参考几个例子：

::

    {
        "id" : 1,
        "name" : "YangWenjian",
        "company" : "Neunn"
    }
    这里使用对象Person接收Json字符串。

::

    [{
        "id" : 1,
        "name" : "YangWenjian",
        "company" : "Neunn"
    }
    {
        "id" : 1,
        "name" : "YangWenjian",
        "company" : "Neunn"
    }]
    这里使用对象的数组Person[]接收Json字符串。

::

    {
    "depatemant" : 
        [{
            "id" : 1,
            "name" : "YangWenjian",
            "company" : "Neunn"
        }
        {
            "id" : 1,
            "name" : "YangWenjian",
        }]
    }
    这里使用包含Person列表的Department对象进行接收字符串，注意首先要解析这个对象的名字，而不是直接解析字段。

在OpenStack-Base层的代码中我使用Jackson进行接收json字符串：

::

    ObjectMapper DEFAULT_MAPPER = new ObjectMapper(); //创建ObjectMapper对象
    DEFAULT_MAPPER.enable(SerializationConfig.Feature.WRAP_ROOT_VALUE);//序列化规则
    DEFAULT_MAPPER.enable(DeserializationConfig.Feature.UNWRAP_ROOT_VALUE);//反序列化规则
    DEFAULT_MAPPER.setPropertyNamingStrategy(PropertyNamingStrategy.CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES);//名称对应规则
    DEFAULT_MAPPER.disable(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES);//反序列化时忽略未定义的属性
    T entity = (T) DEFAULT_MAPPER.readValue(jsonData, returnType);//从json中读取对象，如果json是[]，需要用数组的类型；
    /**也可以用List与Bean对象的组合，此方法已经deprecation
    entity = (T) DEFAULT_MAPPER.readValue(jsonData, TypeFactory.collectionType(List.class,OstfTestSet.class)); */
    

REST缓存机制
-----------------------------------------
REST可以将服务器内容缓存到中间层，减少服务器的访问

.. image:: ../../images/rest_cache1.jpg


构建rest接口
----------------------------------------
使用Jersy框架可以很容易实现rest api和client。

对于client，jersy提供一个Client实现，初始化时加入默认的config，之后使用WebSource类进行目标元素的获取，之后构建一个MultivaluedMap类的header，作为报文的header内容，通过WebSource的handle方法进行请求，并取回相应的response。

请求加密
````````````````````````````````````````
Header中的文字是含有敏感信息的，因此需要我们自己进行加密，否则如果被人截获这个请求，很容易就知道我们的帐号密码或者其他重要信息。

好在javax.crypto包中提供的加密的类及算法。
首先获取Mac类中的一个实例，将待加密的字符串转换成byte数组，利用Mac中的dofinal进行加密，再通过Apache包中的Base64进行encode，最后转化为字符串。
进行认证的时候与加密过程类似，但工序相反，首先通过Base64类进行decode，之后利用Mac类中的dofinal对原始字符串进行加密，之后与参数进行对比得出结果。

之前碰到一点问题就是在windows上跑通的加密算法在linux上并不能验证通过，这是由于linux平台与windows平台在进行encode和decode方式不同导致的。

服务端认证中，采用模板方式，将认证过程实现在父类中，主要算法在父类中实现，其余参数计算等在子类中实现。在服务端用同样方式和密钥进行加密，之后比较客户端发来的加密信息与服务端自己产生的加密信息比较，如果完全相同就返回true表示认证通过。


参考资料
========================================
http://www.infoq.com/cn/articles/understanding-restful-style
