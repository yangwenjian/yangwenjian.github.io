


=======================================
Spring Cloud Netflix
=======================================

Feature
---------------------------------------
* Service Discovery: Eureka instances register and clients discover instances using spring-managed beans.
* Service Discovery: embedded Eureka server.
* Circuit Breaker: Hystrix clients(annotation-driven) and hystrix dashboard.
* Declarative REST Clients.
* Client Side Load Balancer: Ribbon.
* External Configuration: bridge from the Spring Environment.
* Router and Filter: Zuul filters.

Service Discovery: Eureka Clients
---------------------------------------
Eureka is the Netflix Service Discovery Server and Client. The server can be configured and deployed to be highly 
available, with each server replicating state about the registered services to the others.

Registering
```````````````````````````````````````
Client provides meta-data to Eureka(host, port, URL, home page, etc.), Eureka recieve heartbeat.

Automatically reistering. A client can query the registry to locate other services.

Stauts Page and Health Indicator
```````````````````````````````````````
Default page is '/info' and '/health', you can change them.

.. code::

    eureka:
    instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health 





Health Checks
```````````````````````````````````````
Default, client only send 'UP' state. 避免出现网络流量占用过多。

Eureka Metadata
```````````````````````````````````````
There is standard metadata for information such as hostname, IP address, port numbers, the status page, and health check. 
Additional metadata can be added to the instance registration in the eureka.instance.metadataMap, and this metadata is 
accessible in the remote clients.

Using the EurekaClient
```````````````````````````````````````
Do not use the EurekaClient in a @PostConstruct method or in a @Scheduled method.

Spring Cloud has support for Feign (a REST client builder) and Spring RestTemplate through the logical Eureka service 
identifiers (VIPs) instead of physical URLs.

Service Discovery: Eureka Server
--------------------------------------
You can add multiple peers to a system, and, as long as they are all connected to each other by at least one edge, they 
synchronize the registrations amongst themselves.

You can secure your Eureka server simply by adding Spring Security to your server’s classpath via spring-boot-starter-security.

Circuit Breaker: Hystrix Clients
--------------------------------------
A service failure in the lower level of services can cause cascading failure all the way up to the user.
(failure percentage default 50%, rolling window 10 seconds).

The state of the connected circuit breakers are also exposed in the /health endpoint of the calling application,

When using Hystrix commands that wrap Ribbon clients you want to make sure your Hystrix timeout is configured to be longer than 
the configured Ribbon timeout, including any potential retries that might be made.

Client Side Load Balancer: Ribbon
--------------------------------------
Ribbon is a client-side load balancer that gives you a lot of control over the behavior of HTTP and TCP clients. Feign already uses Ribbon.

External Configuration: Archaius
---------------------------------------
Archaius is the Netflix client-side configuration library. It is the library used by all of the Netflix OSS components for configuration. 

Router and Filter: Zuul
---------------------------------------
Zuul is a JVM-based router and server-side load balancer from Netflix.
Add dependency with 'spring-cloud-starter-netflix-zuul'.

* Authentication
* Insights
* Stress Testing
* Canary Testing
* Dynamic Routing
* Service Migration
* Load Shedding
* Security
* Static Response handling
* Active/Active traffic management

the Zuul starter does not include a discovery client, so, for routes based on service IDs, you need to provide one of those on the classpath as well (Eureka is one choice).

.. code::

    zuul:
      routes:
        echo:
          path: /myusers/**
          serviceId: myusers-service
          stripPrefix: true

    hystrix:
      command:
        myusers-service:
          execution:
            isolation:
              thread:
                timeoutInMilliseconds: ...

    myusers-service:
      ribbon:
        NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
        listOfServers: http://example1.com,http://example2.com
        ConnectTimeout: 1000
        ReadTimeout: 3000
        MaxTotalHttpConnections: 500
        MaxConnectionsPerHost: 100

