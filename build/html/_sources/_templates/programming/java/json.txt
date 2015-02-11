


================================
Json Object
================================
Json是JavaScrip Object Notaion的缩写，是轻量级的数据交换格式。
Json非常适合用于网络传输，传出的格式如下：

::

 {
    "name" : "Yang Wenjian",
    "sex" : "male",
    ......
 }

也可以使用带有root name的Json格式：

::

 {
    "Person" : {
        "name" : "Yang Wenjian",
        "sex" : "male",
        ......
    }
 }

我们在构建Restful的接口中经常使用Json作为返回结果，这样可以通过简单的JAVA对象即可接收。

常用的第三方工具包有gson包，还有Jackson-mapper包等。

这里简单比较下使用两个包的一些心得：

gson包简单明了，易于使用，包里类和目录很少，比较容易上手。代码示例如下：

::
 
    Gson gson = new GsonBuilder()
        .excludeFieldsWithModifiers(Modifier.STATIC,Modifier.TRANSIENT, Modifier.VOLATILE)
        .enableComplexMapKeySerialization() // 支持Map的key为复杂对象的形式
        .setPrettyPrinting() // 对json结果格式化
        .setDateFormat("yyyy-MM-dd HH:mm:ss:SSS")// 时间转化为特定格式
        .create();
    T entity = null;
    try {
        entity = (T) gson.fromJson(jsonData, returnType);
    } catch (Exception e) {
        throw new Exception("Error With traslation.");
    }                                                                    

相比之下，JackSon包中的ObjectMapper对于上面第二种带有root name的json序列化或者反序列话支持较好，
有自己封装好的Annotation--@JsonRootName标记在类名之前，易于解析而且不容易出错。代码示例如下：

::

    ObjectMapper DEFAULT_MAPPER = new ObjectMapper();
    DEFAULT_MAPPER.enable(SerializationConfig.Feature.WRAP_ROOT_VALUE);
    DEFAULT_MAPPER.enable(DeserializationConfig.Feature.UNWRAP_ROOT_VALUE);
    DEFAULT_MAPPER.disable(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES);

    T entity = (T) DEFAULT_MAPPER.readValue(jsonData, returnType);

如果想要使用Gson进行带有root name的序列化或者反序列化，需要使用JsonObject。代码示例如下：

::
    
    //序列化对象
    JsonElement je = gson.toJsonTree(object);
    JsonObject jo = new JsonObject();
    jo.add(CostRecordDetails.class.getSimpleName(), je);//类似于一个Mapper，在最外层再加入一个键值对；
    System.out.println(jo.toString());

    //反序列化
    JsonElement je2 = new JsonParser().parse(jsonData);
    JsonElement je3 = je2.getAsJsonObject().get(returnType.getSimpleName());
    T entity = (T) gson.fromJson(je3, returnType);
