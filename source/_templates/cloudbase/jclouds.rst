



=====================================
OpenStack SDK -- JClouds 
=====================================
JClouds is an SDK of common clouds platform.

Introduction
=====================================
JClouds是一个开源框架，它可帮你在云计算中起步并重用你的Java和Clojure开发技能。
JClouds的API允许你自由的使用可迁移的抽象或特定的云特性。我们支持多种云环境：Amazon, VMWare, Azure 和 Rackspace

Excercise
=====================================
We use jclouds as our SDK to build our own cloud system.

JClouds中的请求
-------------------------------------
JClouds作为Openstack的sdk，不外乎的作用就是将用户请求的接口转换为http的请求，然后将返回的json结果转换为结构化的对象。

首先jclouds会以用户名和密码请求相应权限的token，这些是是现在getapi请求中。
之后再利用token请求相应接口，我们看一个示例：

::

    @Named("security-groups:create")                                #方法名称
    @POST                                                           #向发送的http请求类型
    @Path("/ecurity-group-rules")                                   #发送http请求的路径，一般附在endpoint后
    @SelectJson("security_group_rule")                              #返回数据的首元素标志
    @Fallback(NullOnNotFoundOr404.class)                            #返回失败后的处理方式及jclouds的返回结果
    @Consumes(MediaType.APPLICATION_JSON)                           #request接受类型的格式
    @Produces(MediaType.APPLICATION_JSON)                           #response的格式
    @MapBinder(BindSecurityGroupRuleToJsonPayload.class)            #将参数转化为json的类名称
    SecurityGroupRule createSecurityGroupRule(Ingress groupRule);   #接口方法

这里就是整个接口的调用方式，利用java的annotation，使用一套调用模板，可以在切面中加入任意的接口，而不会影响之前的任何代码实现。

这里面，有个地方需要注意，MapBinder是一个接口，是：

  <R extends HttpRequest> R bindToRequest(R request, Map<String, Object> postParams);

转换的类必须实现这个接口，如：

::

    @Singleton
    public class BindSecurityGroupRuleToJsonPayload extends BindToJsonPayload implements MapBinder{
        @Override
        public <R extends HttpRequest> R bindToRequest(R request, Map<String, Object> postParams){
            //获取request，得到参数对象
            Builder<String, Object> payload = ImmutableMap.builder();
            payload.putAll(postParams);
            checkArgument(checkNotNull(request, "request") instanceof GeneratedHttpRequest,
                "this binder is only valid for GeneratedHttpRequests!");
            GeneratedHttpRequest gRequest = (GeneratedHttpRequest) request;
                                             
            Ingress ingress = Ingress.class.cast(find(gRequest.getInvocation().getArgs(), instanceOf(Ingress.class)));
            //之后将所有的属性名称和值放入payload中，这里省略很多重复代码
            if(isNotEmpty(ingress.getProtocol())) {
                payload.put("protocol", ingress.getProtocol());
            }
            return super.bindToRequest(request, ImmutableMap.of("security_group_rule", payload.build()));
        }
    }

具体请求方法
`````````````````````````````````````
具体请求方法在InvokeSyncToAsyncHttpMethod类中，具体方法为：

::

    public Object invoke(Invocation invocation) {
        String commandName = config.getCommandName(invocation);
        HttpCommand command = toCommand(commandName, invocation);
        Function<HttpResponse, ?> transformer = getTransformer(commandName, command);
        org.jclouds.Fallback<?> fallback = getFallback(commandName, invocation, command);
        logger.debug(">> invoking %s", commandName);
        try {
            return transformer.apply(http.invoke(command));
        } catch (Throwable t) {
            try {
                return fallback.createOrPropagate(t);
            } catch (Exception e) {
                throw propagate(e);
            }
        }
    }


JClouds中的token缓存机制
-------------------------------------
JClouds还有一个重要作用，就是将请求的token进行状态缓存。这是为了充分利用token的有效期，并能有效的减少请求次数。

将token请求后进行有效的存储。

::

    V value = null;
    try {
        value = getUninterruptibly(newValue);
        if (value == null) {
            throw new InvalidCacheLoadException("CacheLoader returned null for key " + key + ".");
        }
        statsCounter.recordLoadSuccess(loadingValueReference.elapsedNanos());
        storeLoadedValue(key, hash, loadingValueReference, value);
        return value;
    } finally {
        if (value == null) {
            statsCounter.recordLoadException(loadingValueReference.elapsedNanos());
            removeLoadingValue(key, hash, loadingValueReference);
        }
    }

同时还写了自动刷新token的代码，当请求token的时候，就请求scheduleRefresh方法，这个方法会返回一个有效的token。
这里使用一个时间记录标记now获得当前时间，然后将now值传入scheduleRfresh中，得到一个现在有效的token。

::

    if (count != 0) {
    // read-volatile
    // don't call getLiveEntry, which would ignore loading values
        ReferenceEntry<K, V> e = getEntry(key, hash);
        if (e != null) {
            long now = map.ticker.read();
            V value = getLiveValue(e, now);
            if (value != null) {
                recordRead(e, now);
                statsCounter.recordHits(1);
                return scheduleRefresh(e, key, hash, value, now, loader);
            }
            ValueReference<K, V> valueReference = e.getValueReference();
            if (valueReference.isLoading()) {
                return waitForLoadingValue(e, key, valueReference);
            }
        }
    }


JClouds将值存储之后进行定时刷新，如果时间超过定时刷新时间，就重新获取value并且返回，如果不是，那么就会返回oldValue。

::

    V scheduleRefresh(ReferenceEntry<K, V> entry, K key, int hash, V oldValue, long now,
        CacheLoader<? super K, V> loader) {
        if (map.refreshes() && (now - entry.getWriteTime() > map.refreshNanos)
            && !entry.getValueReference().isLoading()) {
            V newValue = refresh(key, hash, loader, true);
            if (newValue != null) {
                return newValue;
            }
        }
        return oldValue;
    }

JClouds源代码修改
=========================================
明白了JClouds的工作原理后，我们就可以自己添加相应的openstack接口的java sdk（因为jclouds漏了很多，提bug的页面经常不好使，我们期望它在下一个版本修复）

绑定浮动IP
-----------------------------------------
浮动IP的绑定就是将虚拟机赋予一个外网IP，从而在公网上就可以访问，并且可以提供相应服务了。

从网络逻辑上是将外部网络（这里是exnet）中的一个IP赋予与之相连的某个子网中的虚拟机，通过路由器将这个IP的包直接送到虚拟机主机所在网卡，进而送到该虚拟机中。

通过命令：

::

    neutron --debug floatingip-associate FLOATING_IP_ID PORT_ID

显示结果为：

::

    DEBUG: neutronclient.client 
    REQ: 
    curl -i http://192.168.250.222:9696/v2.0/floatingips/10813711-a7ab-4aea-92d6-554dd4f7082b.json
        -X PUT -H "X-Auth-Token:"......" 
        -H "Content-Type: application/json" -H "Accept: application/json" 
        -H "User-Agent: python-neutronclient" 
        -d '{
                "floatingip": {
                    "port_id": "94da9cf4-1948-44ae-b2ae-8fba464aada8"
                }
            }

    RESP: 
        {
            'date': 'Tue,
            16Sep201407: 16: 58GMT',
            'status': '200',
            'content-length': '376',
            'content-type': 'application/json;charset=UTF-8'
        }{
         "floatingip": {
            "router_id": "cbd8cebf-6688-44cb-a4f6-ece233af0108",
            "status": "DOWN",
            "tenant_id": "51b1873935554545bb9b953e4055a997",
            "floating_network_id": "ec8b7335-ee0f-45b1-add5-9b6cafb72660",
            "fixed_ip_address": "192.168.20.200",
            "floating_ip_address": "192.168.252.192",
            "port_id": "94da9cf4-1948-44ae-b2ae-8fba464aada8",
            "id": "10813711-a7ab-4aea-92d6-554dd4f7082b"
        }

根据debug的结果，则相应的API为：

::

    @Path("/v2.0/floatingips")
    @RequestFilters(AuthenticateRequest.class)
    @Consumes(MediaType.APPLICATION_JSON)
    public interface FloatingIPApi {

        /**
        * 为虚拟机绑定浮动IP
        * @parm floatingip_id, port_id
        */
        @Named("floatingip:associate")
        @PUT
        @PATH("/{floatingip_id}")
        @Consumes(MediaType.APPLICATION_JSON)
        @Produces(MediaType.APPLICATION_JSON)
        @Payload("%7B\"floatingip\":%7B\"port_id\":\"{port_id}\"%7D%7D")
        FloatingIP associateIp(@PathParam(floatingip_id) String floatingip_id, @PayloadParam(port_id) String port_id)

        /**
        * 为虚拟机解绑浮动IP
        * @parm floatingip_id
        */
        @Named("floatingip:disassociate")
        @PUT
        @PATH("/{floatingip_id}")
        @Consumes(MediaType.APPLICATION_JSON)
        @Produces(MediaType.APPLICATION_JSON)
        @Payload("%7B\"floatingip\":%7B\"port_id\":null%7D%7D")
        FloatingIP disassociateIp(@PathParam(floatingip_id) String floatingip_id)

    }
