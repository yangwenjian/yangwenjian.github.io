

=======================================================
Cas
=======================================================
Central Authentication Service 是开源的单点登录解决方案。

Cas介绍
=======================================================
支持的协议：Protocols ： Custom Protocol 、 CAS 、 OAuth 、 OpenID 、 RESTful API 、 SAML1.1 、 SAML2.0。
支持的认证机制：Active Directory 、 JAAS 、 JDBC 、 LDAP 、 X.509 Certificates。
安全策略：使用票据（ Ticket ）来实现支持的认证协议；
支持授权：可以决定哪些服务可以请求和验证服务票据（ Service Ticket ）；
提供高可用性：通过把认证过的状态数据存储在 TicketRegistry 组件中，这些组件有很多支持分布式环境的实现，
如： BerkleyDB 、 Default 、 EhcacheTicketRegistry 、 JDBCTicketRegistry 、 JBOSS TreeCache 、 JpaTicketRegistry 、 MemcacheTicketRegistry 等；
支持多种客户端： Java, .Net, PHP, Perl, Apache, uPortal 等。

Cas流程
========================================================
Cas的访问流程分为几个步骤：

1. 用户访问Cas保护的资源时，部署在客户Web应用的Cas AuthenticationFilter，会截获此请求，生成service参数，然后redirect到CAS 服务的login 接口;

.. image:: images/cas_process1.jpg

2. 用户与Cas Server进行交互，进行身份认证，认证成功后，CAS服务器会生成认证cookie，写入浏览器，同时将cookie缓存到服务器本地，
   并为客户端浏览器设置一个 Ticket Granted Cookie（TGC），CAS 服务器还会根据service 参数生成ticket,ticket会保存到服务器，也会加在url后面，然后将请求redirect回客户Web应用；

.. image:: images/cas_process2.jpg

3. Web应用的AuthenticationFilter 看到ticket参数后，会跳过，由其后面的TicketValidationFilter处理，TicketValidationFilter会利用httpclient工具
   访问cas服务的/serviceValidate接口, 将ticket、service都传到此接口，由此接口验证ticket的有效性，TicketValidationFilter 如果得到验证成功的消息，
   就会把用户信息写入web 应用的session里；

.. image:: images/cas_process3.jpg

4. 至此为止，SSO会话就建立起来了，以后用户在同一浏览器里访问此web应用时，AuthenticationFilter会在session里读取到用户信息，所以就不会去CAS认证，
   如果在此浏览器里访问别的web 应用时，AuthenticationFilter在session 里读取不到用户信息，会去CAS 的login接口认证，但这时CAS会读取到浏览器传来的cookie，
   所以CAS不会要求用户去登录页面登录，只是会根据service参数生成一个ticket ，然后再和web应用做一个验证ticket的交互而已。

Cas Filter处理逻辑
=========================================================


最佳实践
=========================================================
用户在一个应用验证通过后，以后用户在同一浏览器里访问此应用时，客户端应用中的过滤器会在 session 里读取到用户信息，
所以就不会去CAS Server认证。如果在此浏览器里访问别的web应用时，客户端应用中的过滤器在 session 里读取不到用户信息，
就会去CAS Server的login接口认证，但这时CAS Server会读取到浏览器传来的cookie （ TGC ），所以CAS Server不会要求用户去登录页面登录，
只是会根据 service参数生成一个ticket，然后再和web应用做一个验证ticket的交互而已。

使用自己的Handler来处理登录登出
---------------------------------------------------------

Spring通过如下配置文件进行配置Cas Client:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    <!-- 声明Cas认证切入点，并声明默认配置为false，以加入自己的定制 -->
    <security:http entry-point-ref="casAuthenticationEntryPoint" auto-config="false">
        <!-- 声明被保护的资源，注意顺序，并加入spring security的权限管理 -->
        <security:intercept-url pattern="/checkauthority.do" access="IS_AUTHENTICATED_ANONYMOUSLY" />
        <security:intercept-url pattern="/\**/\*.cas" access="ROLE_USER,ROLE_DOCTOR" />

        <!-- 如果声明默认配置为true，可以仅指定logout-success-url，其余都有默认初始值 --> 
        <security:logout logout-success-url="${cas.securityContext.casProcessingFilterEntryPoint.logoutUrl}?service=${index.url}" /> -->
        <security:custom-filter ref="concurrencyFilter" position="CONCURRENT_SESSION_FILTER" />
        <security:custom-filter ref="casAuthenticationFilter" position="CAS_FILTER"/>
        <security:custom-filter ref="singleLogoutFilter" before="CAS_FILTER"/>
        <security:custom-filter ref="requestSingleLogoutFilter" position="LOGOUT_FILTER"/>
    </security:http>

    <bean id="casAuthenticationEntryPoint" class="org.springframework.security.cas.web.CasAuthenticationEntryPoint">
        <property name="loginUrl" value="${cas.securityContext.casProcessingFilterEntryPoint.loginUrl}"/>
        <property name="serviceProperties" ref="serviceProperties"></property>
    </bean>

    <bean id="serviceProperties" class="org.springframework.security.cas.ServiceProperties">
        <property name="service" value="${cas.securityContext.serviceProperties.service}" />
        <property name="sendRenew" value="false" />
    </bean>

    <security:authentication-manager alias="authenticationManager">
        <security:authentication-provider ref="casAuthenticationProvider"/>
    </security:authentication-manager>

    <bean id="casAuthenticationProvider" class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
        <property name="authenticationUserDetailsService" ref="authenticationUserDetailsService" />
        <property name="serviceProperties" ref="serviceProperties"></property>
        <property name="ticketValidator">
            <bean class="org.jasig.cas.client.validation.Cas20ServiceTicketValidator">
                <constructor-arg index="0" value="${cas.securityContext.ticketValidator.casServerUrlPrefix}"></constructor-arg>
            </bean>
        </property>
        <property name="key" value="an_id_for_this_auth_provider_only"></property>
    </bean>

    <bean id="casAuthenticationFilter" class="org.springframework.security.cas.web.CasAuthenticationFilter">
        <property name="authenticationManager" ref="authenticationManager"/> 
        <property name="authenticationSuccessHandler" ref="authenticationSuccessHandler"/>
    </bean>

    <bean id="authenticationSuccessHandler" class="com.xikang.ch.cas.MyAuthenticationSuccessHandler">
        <property name="alwaysUseDefaultTargetUrl" value="true" />
        <property name="defaultTargetUrl" value="${index.url}" />
        <property name="serverName" value="${ch.domain}" />
    </bean>

    <bean id="concurrencyFilter" class="org.springframework.security.web.session.ConcurrentSessionFilter">  
        <property name="sessionRegistry" ref="sessionRegistry" />  
        <property name="expiredUrl" value="${cas.securityContext.casProcessingFilterEntryPoint.logoutUrl}" />  
    </bean> 

    <bean id="sessionRegistry" class="org.springframework.security.core.session.SessionRegistryImpl" />

    <bean id="authenticationUserDetailsService" class="com.xikang.ch.cas.GrantedAuthorityFromAssertionAttributesXKUserDetailsService">
        <constructor-arg>
            <array>
                <value>authorities</value>
            </array>
        </constructor-arg>
    </bean>
    <bean id="proxyGrantingTicketStorage" class="org.jasig.cas.client.proxy.ProxyGrantingTicketStorageImpl" />

    <!--登出配置-->

    <bean id="singleLogoutFilter" class="org.jasig.cas.client.session.SingleSignOutFilter"/>

    <bean id="requestSingleLogoutFilter" class="org.springframework.security.web.authentication.logout.LogoutFilter">
        <constructor-arg value="${cas.securityContext.casProcessingFilterEntryPoint.logoutUrl}" />
        <constructor-arg>
            <!-- <bean class="org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler" /> -->
            <bean class="com.xikang.cn.cas.MySecrityContextLogouthandler"/>
        </constructor-arg>
        <property name="filterProcessesUrl" value="/j_spring_security_logout" />
    </bean>

Spring cas关键代码
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

org.springframework.security.web.authentication.AbstractrAuthenticationProcessingFilter:

.. code:: java

    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        if (!requiresAuthentication(request, response)) {
            chain.doFilter(request, response);
            return;
        }
        if (logger.isDebugEnabled()) {
            logger.debug("Request is to process authentication");
        }
        Authentication authResult;
        try {
            authResult = attemptAuthentication(request, response);
            if (authResult == null) {
                // return immediately as subclass has indicated that it hasn't completed authentication
                return;
            }
            sessionStrategy.onAuthentication(authResult, request, response);
        }catch(InternalAuthenticationServiceException failed) {
            logger.error("An internal error occurred while trying to authenticate the user.", failed);
            unsuccessfulAuthentication(request, response, failed);
            return;
        }catch (AuthenticationException failed) {
            unsuccessfulAuthentication(request, response, failed);
            return;
        }
        //Authentication success
        if (continueChainBeforeSuccessfulAuthentication) {
            chain.doFilter(request, response);
        }

        successfulAuthentication(request, response, chain, authResult);
    }

    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
                Authentication authResult) throws IOException, ServletException{
        successfulAuthentication(request, response, authResult);
    }
    
    @Deprecated
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response,
                Authentication authResult) throws IOException, ServletException {
        if (logger.isDebugEnabled()) {
            logger.debug("Authentication success. Updating SecurityContextHolder to contain: " + authResult);
        }
        SecurityContextHolder.getContext().setAuthentication(authResult);
        rememberMeServices.loginSuccess(request, response, authResult);
        if (this.eventPublisher != null) {
            eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
        }
        successHandler.onAuthenticationSuccess(request, response, authResult);
    }

org.springframework.security.web.authentication.logout.LogoutFilter:

.. code:: java

    public LogoutFilter(String logoutSuccessUrl, LogoutHandler... handlers) {
        Assert.notEmpty(handlers, "LogoutHandlers are required");
        this.handlers = Arrays.asList(handlers);
        Assert.isTrue(!StringUtils.hasLength(logoutSuccessUrl) || UrlUtils.isValidRedirectUrl(logoutSuccessUrl), logoutSuccessUrl + " isn't a valid redirect URL");
        SimpleUrlLogoutSuccessHandler urlLogoutSuccessHandler = new SimpleUrlLogoutSuccessHandler();
        if (StringUtils.hasText(logoutSuccessUrl)) {
            urlLogoutSuccessHandler.setDefaultTargetUrl(logoutSuccessUrl);
        }
        logoutSuccessHandler = urlLogoutSuccessHandler;
        setFilterProcessesUrl("/j_spring_security_logout");
    }
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        if (requiresLogout(request, response)) {
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (logger.isDebugEnabled()) {
                logger.debug("Logging out user '" + auth + "' and transferring to logout destination");
            }
            for (LogoutHandler handler : handlers) {
                handler.logout(request, response, auth);
            }
            logoutSuccessHandler.onLogoutSuccess(request, response, auth);
            return;
        }
        chain.doFilter(request, response);
    }


参考资料
===========================================================
https://www.ibm.com/developerworks/cn/opensource/os-cn-cas/

