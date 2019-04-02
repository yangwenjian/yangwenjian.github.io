

=======================================
Spring Cloud
=======================================
Main Framework of building micro-service based on java.

Introduction
=======================================
Spring Cloud is based on Spring boot.

微服务
=======================================

微服务基本原则：

* 单一目的 - 每项服务应专注于一个目的并做好。
* 松耦合 - 服务彼此间没有太多联系。更改一项服务时不应要求更改其他服务。应仅通过公共服务接口进行服务之间的通信。
* 高内聚 - 每个服务将所有相关行为和数据封装在一起。如果我们需要构建新功能，所有更改仅限于一个服务。

缺少单一目的，每个微服务最终会做太多事情，成长为多个“单体”服务。我们不会从微服务架构中获得全部好处，还要支付运营成本。

缺少松耦合，对一项服务的更改会影响其他服务，因此我们无法快速安全地发布更改，这是微服务架构的核心优势。更重要的是，紧耦合引起的问题可能是灾难性的，例如数据不一致甚至数据丢失。

缺少高内聚，我们将最终得到一个分布式的单体系统，一组混乱的服务，必须同时进行更改和部署才能构建单一功能。由于多个服务协调的复杂性和成本（有时跨多个团队），分布式单体系统通常比集中式单体系统差得多。

与此同时，了解微服务不是什么也很重要：

微服务不是代码行数少或处理“微”任务的服务。这种误解来自“ 微服务” 这个名称。微服务架构的目标不是拥有尽可能多的小型服务。只要符合上述三项原则，服务也能够是复杂而重要的。

微服务并不总是由新技术构建。尽管微服务架构能够让团队更轻松地验证新技术，但它并不是微服务架构的主要目标。只要团队从分离的服务中受益，就可以使用完全相同的技术栈构建新服务。

微服务不是必须从头开始构建的服务。如果你已经拥有一个架构良好的单体应用程序，请避免养成从头开始构建每个新服务的习惯。也可以直接从单体服务中直接提取逻辑。同样，应该仍然遵循上述三个原则。

Feature
---------------------------------------
* Distributed/versioned configuration
* Service registration and discovery
* Routing
* Service-to-service calls
* Load balancing
* Circuit Breakers
* Global locks
* Leadership election and cluster state
* Distributed messaging

Main Project
---------------------------------------

Spring Cloud Config
```````````````````````````````````````
Centralized external configuration management backed by a git repository. 类似CMDB配置管理数据库抽象服务。

Spring Cloud Netflix
```````````````````````````````````````
Integration with various Netflix OSS components (Eureka, Hystrix, Zuul, Archaius, etc.).

Spring Cloud Bus
```````````````````````````````````````
An event bus, distributed messaging.

Spring Cloud for Cloud foundry
```````````````````````````````````````
Integrates with Pivotal Cloud Foundry.

Spring Cloud Cluster
```````````````````````````````````````
Leadership election and common stateful patterns.(Zookeeper, Redis, hazelcast, Consul).

Spring Cloud Open Service Broker
```````````````````````````````````````
Provides a starting point for building a service broker that implements the Open Service Broker API.

Spring Cloud Consul
```````````````````````````````````````
Service discovery and configuration management with Hashicorp Consul.

Spring Cloud Security
```````````````````````````````````````
Provides support for load-balanced OAuth2 rest client and authentication header relays in a Zuul proxy.

Spring Cloud Sleuth
```````````````````````````````````````
Distributed tracing for Spring Cloud applications, compatible with Zipkin, HTrace and log-based (e.g. ELK) tracing.

Spring Cloud Data Flow
```````````````````````````````````````
A cloud-native orchestration service for composable microservice applications on modern runtimes. Easy-to-use DSL, 
drag-and-drop GUI, and REST-APIs together simplifies the overall orchestration of microservice based data pipelines.

Spring Cloud Stream
````````````````````````````````````````
A lightweight event-driven microservices framework to quickly build applications that can connect to external systems. 
Simple declarative model to send and receive messages using Apache Kafka or RabbitMQ between Spring Boot apps.

Spring Cloud Stream App Starters
````````````````````````````````````````
Spring Cloud Stream App Starters are Spring Boot based Spring Integration applications that provide integration with 
external systems.

Spring Cloud Task
```````````````````````````````````````
A short-lived microservices framework to quickly build applications that perform finite amounts of data processing. 
Simple declarative for adding both functional and non-functional features to Spring Boot apps.

Spring Cloud Task App Starters
```````````````````````````````````````
Spring Cloud Task App Starters are Spring Boot applications that may be any process including Spring Batch jobs that 
do not run forever, and they end/stop after a finite period of data processing.

Spring Cloud Zookeeper
````````````````````````````````````````
Service discovery and configuration management with Apache Zookeeper.

Spring Cloud for Amazon Web Services
````````````````````````````````````````
Easy integration with hosted Amazon Web Services. It offers a convenient way to interact with AWS provided services 
using well-known Spring idioms and APIs, such as the messaging or caching API. Developers can build their application 
around the hosted services without having to care about infrastructure or maintenance.

Spring Cloud Connectors
```````````````````````````````````````
Makes it easy for PaaS applications in a variety of platforms to connect to backend services like databases and message 
brokers (the project formerly known as "Spring Cloud").

Spring Cloud Starters
```````````````````````````````````````
Spring Boot-style starter projects to ease dependency management for consumers of Spring Cloud. (Discontinued as a project 
and merged with the other projects after Angel.SR2.)

Spring Cloud CLI
````````````````````````````````````````
Spring Boot CLI plugin for creating Spring Cloud component applications quickly in Groovy.

Spring Cloud Contract
````````````````````````````````````````
Spring Cloud Contract is an umbrella project holding solutions that help users in successfully implementing the Consumer 
Driven Contracts approach.

Spring Cloud Gateway
```````````````````````````````````````
Spring Cloud Gateway is an intelligent and programmable router based on Project Reactor.

Spring Cloud OpenFeign
````````````````````````````````````````
Spring Cloud OpenFeign provides integrations for Spring Boot apps through autoconfiguration and binding to the Spring Environment 
and other Spring programming model idioms.

Spring Cloud Pipelines
````````````````````````````````````````
Spring Cloud Pipelines provides an opinionated deployment pipeline with steps to ensure that your application can be deployed in 
zero downtime fashion and easilly rolled back of something goes wrong.

Spring Cloud Function
````````````````````````````````````````
Spring Cloud Function promotes the implementation of business logic via functions. It supports a uniform programming model across 
serverless providers, as well as the ability to run standalone (locally or in a PaaS).

个人理解
```````````````````````````````````````
* 个人认为，如下几个服务是必要的：服务的自动注册和发现，负载均衡
* 大型系统同时还需要：服务监控，服务链路跟踪，融断器
* 外网服务的系统还需要：网关服务，认证服务


Spring Cloud Gateway
=======================================
Spring Cloud Gateway requires the Netty runtime provided by Spring Boot and Spring Webflux. It does not work in a traditional Servlet Container or built as a WAR.

Glossary
---------------------------------------
* Route: route the basic building block.
* Predicate: match on anything from the HTTP request.
* Filter: requests and responses can be modified before or after sending the downstream request.

How it works
---------------------------------------
Gateway Client -> Gateway Handler Mapping(determines match route or not) -> Gateway Web Handler(send to filter) -> Filter chain(execute logic 
before the proxy request is sent or after) -> Proxied Service -> Filter chain().

.. image:: images/spring_cloud_gateway_diagram.png

Route Predicate Factories
----------------------------------------
Many build-in Route Predicate Factories match on different attributes of the HTTP request.

* After Route Predicate Factory(datetime)
* Before Route Predicate Factory
* Between Route Predicate Factory
* Cookie Route Predicate Factory(cookie name and regular expression value)
* Header Route Predicate Factory(header name and regular expression value)
* Method Route Predicate Factory(GET, POST)
* Path Route Predicate Factory
* Query Route Predicate Factory
* RemoteAddr Route Predicate Factory

If RemoteAddr Route does not work because Gateway sits behind a proxy layer, use RemoteAddressResolve instead.

XForwardedRemoteAddressResolver::trustAll check IP in X-Forwared-For;

XForwardedRemoteAddressResolver::maxTrustedIndex check numbers of infrastructure in front of Gateway.


GatewayFilter Factories
-----------------------------------------
Route filters allow the modification of the incoming HTTP request or outgoing HTTP response in some manner. 

Spring Cloud Gateway includes many built-in GatewayFilter Factories.只要添加适当的配置即可使用。

* AddRequestHeader GatewayFilter Factory(all matching request)
* AddRequestParameter GatewayFilter Factory
* AddResponseHeader GatewayFilter Factor
* Hystrix GatewayFilter Factory(add dependency)

you can use '- Hystrix=myCommandName' or fallbackuri.

.. code::

    spring:
        cloud:
        gateway:
        routes:
        - id: hystrix_route
            uri: lb://backing-service:8088
            predicates:
            - Path=/consumingserviceendpoint
            filters:
            - name: Hystrix
            args:
                name: fallbackcmd
                fallbackUri: forward:/incaseoffailureusethis
            - RewritePath=/consumingserviceendpoint, /backingserviceendpoint       

* PrefixPath GatewayFilter Factory(add prefixpath)
* PreserveHostHeader GatewayFilter Factory(set original host header)
* RequestRateLimiter GatewayFilter Factory
* RedirectTo GatewayFilter Factory
* RemoveNonProxyHeaders GatewayFilter Factory
* RemoveRequestHeader GatewayFilter Factory
* RemoveResponseHeader GatewayFilter Factory
* RewritePath GatewayFilter Factory
* SaveSession GatewayFilter Factory(Spring Sesion 必备)
* SetPath GatewayFilter Factory
* SetResponseHeader GatewayFilter Factory
* SetStatus GatewayFilter Factory
* StripPrefix GatewayFilter Factory
* Retry GatewayFilter Factory

Redis RateLimiter
```````````````````````````````````````````
The algorithm used is the Token Bucket Algorithm.

.. code::

    spring:
      cloud:
        gateway:
          routes:
          - id: requestratelimiter_route
            uri: http://example.org
            filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10 #桶的回填速率
                redis-rate-limiter.burstCapacity: 20 #桶的回填速率


Global Filters
--------------------------------------------
The GlobalFilter interface has the same signature as GatewayFilter.

This combined filter chain is sorted by the org.springframework.core.Ordered interface, which can be set by implementing 
the getOrder() method or by using the @Order annotation.

* Forward Routing Filter
* LoadBalancerClient Filter
* Netty Routing Filter
* Netty Write Response Filter
* RouteToRequestUrl Filter
* Websocket Routing Filter
* Gateway Metrics Filter

Websocket Routing
--------------------------------------------
可以通过配置文件或者代码中builder的构建指定websocket的路由。

.. code::

   spring:
      cloud:
        gateway:
          routes:
          # SockJS route
          - id: websocket_sockjs_route
            uri: http://localhost:3001
            predicates:
            - Path=/websocket/info/**
          # Normwal Websocket route
          - id: websocket_route
            uri: ws://localhost:3001
            predicates:
            - Path=/websocket/** 

TLS/SSL
---------------------------------------------
Gateway routes can be routed to both http and https backends.

.. code::

    server:
      ssl:
        enabled: true
        key-alias: scg
        key-store-password: scg1234
        key-store: classpath:scg-keystore.p12
        key-store-type: PKCS12

    spring:
      cloud:
        gateway:
          httpclient:
            ssl:
              trustedX509Certificates:
              - cert1.pem
              - cert2.pem

CORS Configuration
---------------------------------------
.. code::

    spring:
      cloud:
        gateway:
          globalcors:
            corsConfigurations:
              '[/\**]':
                allowedOrigins: "docs.spring.io"
                allowedMethods:
                - GET

Using Spring MVC or Webflux
---------------------------------------
Spring Cloud Gateway provides a utility object called ProxyExchange which you can use inside a regular Spring web handler as a method parameter.

.. code::

    //spring-cloud-gateway-mvc:
    @RestController
    @SpringBootApplication
    public class GatewaySampleApplication {

        @Value("${remote.home}")
        private URI home;

        @GetMapping("/test")
        public ResponseEntity<?> proxy(ProxyExchange<byte[]> proxy) throws Exception {
            return proxy.uri(home.toString() + "/image/png").get();
        }

    }

    //spring-cloud-gateway-weflux:
    @RestController
    @SpringBootApplication
    public class GatewaySampleApplication {

        @Value("${remote.home}")
        private URI home;

        @GetMapping("/test")
        public Mono<ResponseEntity<?>> proxy(ProxyExchange<byte[]> proxy) throws Exception {
            return proxy.uri(home.toString() + "/image/png").get();
        }

    }


最佳实践
---------------------------------------
自定义filter

.. code:: java

    @Service
    public class AuthFilter implements GlobalFilter {

        private String remoteOauthServer = "http://172.27.63.11:8080/oauth/check_token?token=";
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            System.out.println("进入AuthFilter");
            boolean isAllow = checkOauthToken(exchange.getRequest());
            if (isAllow) {
                return chain.filter(exchange);
            } else {
            //设置status和body
            return Mono.defer(() -> {
                setResponseStatus(exchange, HttpStatus.UNAUTHORIZED);
                final ServerHttpResponse response = exchange.getResponse();
                byte[] bytes = "Token invalidate".getBytes(StandardCharsets.UTF_8);
                DataBuffer buffer = exchange.getResponse().bufferFactory().wrap(bytes);
                return response.writeWith(Flux.just(buffer));
                });
            }
        }
    }  
