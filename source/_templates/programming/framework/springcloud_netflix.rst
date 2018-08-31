


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

Custom Zuul Filter Examples
`````````````````````````````````````````

.. code:: java

    public class QueryParamPreFilter extends ZuulFilter {
        @Override
        public int filterOrder() {
            return PRE_DECORATION_FILTER_ORDER - 1; // run before PreDecoration
        }
        @Override
        public String filterType() {
            return PRE_TYPE;
        }
        @Override
        public boolean shouldFilter() {
            RequestContext ctx = RequestContext.getCurrentContext();
            return !ctx.containsKey(FORWARD_TO_KEY) // a filter has already forwarded
                    && !ctx.containsKey(SERVICE_ID_KEY); // a filter has already determined serviceId
        }
        @Override
        public Object run() {
            RequestContext ctx = RequestContext.getCurrentContext();
            HttpServletRequest request = ctx.getRequest();
            if (request.getParameter("sample") != null) {
                // put the serviceId in `RequestContext`
                ctx.put(SERVICE_ID_KEY, request.getParameter("foo"));
            }
            return null;
        }
    }

    public class OkHttpRoutingFilter extends ZuulFilter {
        @Autowired
        private ProxyRequestHelper helper;

        @Override
        public String filterType() {
            return ROUTE_TYPE;
        }

        @Override
        public int filterOrder() {
            return SIMPLE_HOST_ROUTING_FILTER_ORDER - 1;
        }

        @Override
        public boolean shouldFilter() {
            return RequestContext.getCurrentContext().getRouteHost() != null
                    && RequestContext.getCurrentContext().sendZuulResponse();
        }

        @Override
        public Object run() {
            OkHttpClient httpClient = new OkHttpClient.Builder()
                    // customize
                    .build();

            RequestContext context = RequestContext.getCurrentContext();
            HttpServletRequest request = context.getRequest();

            String method = request.getMethod();

            String uri = this.helper.buildZuulRequestURI(request);

            Headers.Builder headers = new Headers.Builder();
            Enumeration<String> headerNames = request.getHeaderNames();
            while (headerNames.hasMoreElements()) {
                String name = headerNames.nextElement();
                Enumeration<String> values = request.getHeaders(name);

                while (values.hasMoreElements()) {
                    String value = values.nextElement();
                    headers.add(name, value);
                }
            }

            InputStream inputStream = request.getInputStream();

            RequestBody requestBody = null;
            if (inputStream != null && HttpMethod.permitsRequestBody(method)) {
                MediaType mediaType = null;
                if (headers.get("Content-Type") != null) {
                    mediaType = MediaType.parse(headers.get("Content-Type"));
                }
                requestBody = RequestBody.create(mediaType, StreamUtils.copyToByteArray(inputStream));
            }

            Request.Builder builder = new Request.Builder()
                    .headers(headers.build())
                    .url(uri)
                    .method(method, requestBody);

            Response response = httpClient.newCall(builder.build()).execute();

            LinkedMultiValueMap<String, String> responseHeaders = new LinkedMultiValueMap<>();

            for (Map.Entry<String, List<String>> entry : response.headers().toMultimap().entrySet()) {
                responseHeaders.put(entry.getKey(), entry.getValue());
            }

            this.helper.setResponse(response.code(), response.body().byteStream(),
                    responseHeaders);
            context.setRouteHost(null); // prevent SimpleHostRoutingFilter from running
            return null;
        }
    }
