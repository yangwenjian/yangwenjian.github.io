


================================================
Spring Security
================================================
Spring Security基于Spring框架，提供了一套Web应用安全性的完整解决方案。
包括Web应用的用户认证（Authentication）和用户授权（Authorization）两个部分。

Spring Security架构
================================================

Authentication and Access Control
------------------------------------------------
An AuthenticationManager can do one of 3 things in its authenticate() method:

1. return an Authentication (normally with authenticated=true) if it can verify that the input represents a valid principal.
2. throw an AuthenticationException if it believes that the input represents an invalid principal.
3. return null if it can’t decide.

AuthenticationProvider is a bit like an AuthenticationManager but it has an extra method to allow the caller to query if it 
supports a given Authentication type.


Spring Security如何控制权限
================================================
Spring Security使用Filter组成的Chain来判断权限，主要是有AccessDecisionManager中的AccessDecisionVoter(AuthenticationVoter, 
RoleVoter)进行投票，如果有任何一个voter没有通过，就会抛出AccessDenyException，进行重新登录。

Spring预定义了很多out-of-boxed filter供开发者直接使用。
每个Filter一般情况下（有些Filter是abstract的）都和配置文件的一个元素（有的情况下可能是属性）对应。
比如：AUTHENTICATION_PROCESSING_FILTER，对应配置文件里面的：http/form-login元素。
如果Spring提供的Filter不能满足权限功能，开发者可以自己定义Filter，然后放到Filter Chain的某个位置，或者替换原有的Filter，也
可以放在某个Filter的前面或者后面。

控制内容
------------------------------------------------

- URL
    - 可以分为需要权限判断的url，不需要权限判断的url，登录表单url。需要权限判断的url，仅限于做角色判断，就是说判断当前用户是否具有指定的角色。
- Bean Method
    - Annotation. @SURITY("ROLE_TELLER");
    - xml. <protect method="set" access="ROLE_ADMIN"/>
- HttpSession
    - 控制用户是否能重复登录，及重复登录次数，并非重试密码次数。
- 领域对象
    - 主要通过ACL（访问控制列表）实现。

控制方式
-----------------------------------------------
主要有以下两种方式：

1. 全部利用配置文件，将用户、权限、资源（url）硬编码在xml文件中。
2. 细分角色和权限，并将用户、角色、权限和资源均采用数据库存储，并且自定义过滤器，代替原有的FilterSecurityInterceptor过滤器，并分别实现AccessDecisionManager、InvocationSecurityMetadataSourceService和UserDetailsService，并在配置文件中进行相应配置。

自定义授权管理
-----------------------------------------------

- AbstractSecrurityInterceptor
    - 具体实现类作为过滤器，该过滤器要插入到授权之前。在执行 doFilter 之前，进行权限的检查，而具体的实现交给 accessDecisionManager。
- FilterInvocationSecurityMetadataSource
    - 具体实现类在初始化时，要实现从数据库或其它存储库中加载所有资源与权限（角色），并装配到MAP <String, Collection<ConfigAttribute>>中。 资源通常为url，权限就是那些以ROLE_为前缀的角色，资源为key， 权限为value。 一个资源可以由多个权限来访问。
- UserDetailService
    - 具体实现类从存储库中读取特定用户的各种信息（用户的密码，角色信息，是否锁定，账号是否过期等）。唯一要实现的方法： public UserDetails loadUserByUsername(String username)。
- AccessDecisionManager
    - 匹配权限以决定是否放行。主要实现方法：public void decide (Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes).
        - In this method, need to compare authentication with configAttributes.
        - A object is a URL, a filter was find permission configuration by this URL, and pass to here.
        - Check authentication has attribute in permission configuration (configAttributes).
        - If not match corresponding authentication, throw a AccessDeniedException.

最佳实践
===============================================
配置方式
-----------------------------------------------
1. 使用最简单的配置方式，直接写在xml文件中；
2. 使用dataSource进行配置，数据库中需要有相应的表进行对应；
3. 提供表和查询sql，由Spring Security自己进行查询；
4. 自定义filter和voter等，进行扩展；

参考资料
===============================================
Spring Security3.0.1中文官方文档
http://www.coin163.com/java/docs/201305/d_2785029006.html
