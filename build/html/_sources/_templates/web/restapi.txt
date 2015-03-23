


==========================================
Rest API 基础
==========================================
REST: Representational State Transfor是当前比较流行技术，可以降低耦合度、复杂性，并且提高兼容性，可移植性。

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

RestAPI的请求与返回
========================================
