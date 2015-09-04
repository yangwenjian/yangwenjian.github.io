

=========================================
cas
=========================================
Central Authentication Service 是开源的单点登录解决方案。
支持的协议：Protocols ： Custom Protocol 、 CAS 、 OAuth 、 OpenID 、 RESTful API 、 SAML1.1 、 SAML2.0。
支持的认证机制：Active Directory 、 JAAS 、 JDBC 、 LDAP 、 X.509 Certificates。
安全策略：使用票据（ Ticket ）来实现支持的认证协议；
支持授权：可以决定哪些服务可以请求和验证服务票据（ Service Ticket ）；
提供高可用性：通过把认证过的状态数据存储在 TicketRegistry 组件中，这些组件有很多支持分布式环境的实现，
如： BerkleyDB 、 Default 、 EhcacheTicketRegistry 、 JDBCTicketRegistry 、 JBOSS TreeCache 、 JpaTicketRegistry 、 MemcacheTicketRegistry 等；
支持多种客户端： Java 、 .Net 、 PHP 、 Perl 、 Apache, uPortal 等

最佳实践
=========================================
Cas Client以filter的方式加入的web应用中，CAS Client 会分析 HTTP 请求中是否包含请求 Service Ticket，
如果没有，则说明该用户是没有经过认证的；于是 CAS Client 会重定向用户请求到 CAS Server，并传递 Service。
户认证过程，如果用户提供了正确的 Credentials ， CAS Server 随机产生一个相当长度、唯一、不可伪造的 Service Ticket ，
并缓存以待将来验证，并且重定向用户到 Service 所在地址（附带刚才产生的 Service Ticket ） , 
并为客户端浏览器设置一个 Ticket Granted Cookie（TGC）； 
CAS Client 在拿到 Service 和新产生的 Ticket 过后，与 CAS Server 进行身份核实，以确保 Service Ticket 的合法性。

用户在一个应用验证通过后，以后用户在同一浏览器里访问此应用时，客户端应用中的过滤器会在 session 里读取到用户信息，所以就不会去 CAS Server 认证。如果在此浏览器里访问别的 web 应用时，客户端应用中的过滤器在 session 里读取不到用户信息，就会去 CAS Server 的 login 接口认证，但这时 CAS Server 会读取到浏览器传来的 cookie （ TGC ），所以 CAS Server 不会要求用户去登录页面登录，只是会根据 service 参数生成一个 Ticket ，然后再和 web 应用做一个验证 ticket 的交互而已

参考资料
=========================================
https://www.ibm.com/developerworks/cn/opensource/os-cn-cas/



zookeeper
=========================================
Zookeeper作为一个分布式的服务框架，
主要用来解决分布式集群中应用系统的一致性问题，它能提供基于类似于文件系统的目录节点树方式的数据存储，
但是Zookeeper并不是用来专门存储数据的，它的作用主要是用来维护和监控你存储的数据的状态变化。
通过监控这些数据状态的变化，从而可以达到基于数据的集群管理。
