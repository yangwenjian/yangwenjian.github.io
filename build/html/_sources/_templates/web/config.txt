

===============================================
Web工程配置
===============================================


Web工程记载配置文件
===============================================
context-param用来描述webapp本身的属性，而且在有ServletContext能取值，在servlets、filters、listeners中其配置作用。

利用context-param，加载spring配置文件：

.. code::

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:conf/\*\*/spring-context\*.xml</param-value>
    </context-param>

上面是Web工程能识别的字段，为contextConfigLocation，如果想自定义字段，需要添加读取配置的代码：

.. code:: java
    
    //在contextListener中
    String value = servletConfig.getServletContext().getInitParameter("myParam");
    //在Servlet中
    String value = request.getSession().getServletContext().getInitParameter("myParam");

加载环境变量
------------------------------------------------
Environment entries are available via JNDI which may be useful when you don't have a ServletContext directly at hands,
such as in EJBs. The one in the web.xml is actually the last in precedence chain as to overridding environment entires.

.. code::

    <env-entry> 
        <env-entry-name>dbhost</env-entry-name>
        <env-entry-type>java.lang.String</env-entry-type>
        <env-entry-value>localhost</env-entry-value> 
    </env-entry>

.. code:: java

    // Get the base naming context
    Context env = (Context)new InitialContext().lookup("java:comp/env");    
    // Get a single value
    String dbhost = (String)env.lookup("dbhost");

Web工程启动初始化
================================================
Web容器的线程启动listerner:

.. code::

	<listener>
		<listener-class>com.xikang.ch.core.web.listener.SiteConfigureListener</listener-class>
	</listener>
	<listener>
		<listener-class>com.xikang.ch.portal.system.config.CloudHospitalListenerConfigurator</listener-class>
	</listener>

Filters
================================================
Filter的作用是在servlet接收前对其进行预处理，在response返回给客户端之前也可以进行头和数据的处理。
一个filter可以拦截多个请求，一个请求也能被多个filter拦截。

Filter的执行顺序是由web.xml中filter-mapping的书写顺序决定的。

Spring Security 配置：

.. code::

    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>

In Spring Security, the filter classes are also Spring beans defined in the application context and thus able to take advantage
of Spring's rich dependency-injection facilities and lifecycle interfaces. Spring's DelegatingFilterProxy provides the link 
between web.xml and the application context.

.. code:: java

    @Override
    protected void initFilterBean() throws ServletException {
        synchronized (this.delegateMonitor) {
            if (this.delegate == null) {
                // If no target bean name specified, use filter name.
                if (this.targetBeanName == null) {
                    this.targetBeanName = getFilterName();
                }
                // Fetch Spring root application context and initialize the delegate early,
                // if possible. If the root application context will be started after this
                // filter proxy, we'll have to resort to lazy initialization.
                WebApplicationContext wac = findWebApplicationContext();
                if (wac != null) {
                    this.delegate = initDelegate(wac);
                }
            }
        }
    }

如果使用默认配置，targetBeanName为springSecurityFilterChain，web上下文按web.xml加载进来。



url-pattern
================================================
当请求发送到servlet容器的时候，容器先会将请求的url减去当前应用上下文的路径作为servlet的映射url，比如我访问的是
http://localhost/test/aaa.html， 我的应用上下文是test，容器会将http://localhost/test去掉， 剩下的/aaa.html部分拿来做servlet的
映射匹配。这个映射匹配过程是有顺序的，而且当有一个servlet匹配成功以后，就不会去理会剩下 的servlet了（filter不同，后文会提到）。

匹配类型：

1. 结尾为'/\*';
2. 开头为'\*.';
3. 为'/';
4. 其他精确匹配;

url-pattern遵循如下规则：
1. 精确路径匹配，使用contextVersion的exactWrappers;
2. 前缀匹配，使用contextVersion的wildcardWrappers;
3. 扩展匹配，使用contextVersion的extensionWrappers;
4. 使用资源文件来处理servlet，使用contextVersion的welcomeResources属性，这个属性是个字符串数组;
5. 使用默认的servlet，使用contextVersion的defaultWrapper;
