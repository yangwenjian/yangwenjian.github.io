


=====================================
Spring
=====================================

Spring是当前最主流的企业应用框架

Spring的配置
====================================

Spring的数据源管理
-------------------------------------
Spring在使用Hibernate的时候需要进行初始化配置，建立数据源：
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">（这里使用阿里的数据库驱动）
之后进行数据库连接dataSource的各项配置：driverClassName, url(pay attention to encoding), username, password, initialSize, maxActive(最大活跃数), maxIdle(最大空闲值), minIdle(最小空闲值)。

Spring能自己管理数据连接池，但有时代码不小心也会出现连接泄漏的情况，这里碰到的问题时访问base层一段时间后出现：
卡在Obtaining JDBC Connection，之后超时连接错误。
目前的应对方法是在ApplicationContext.xml文件中加入如下配置：

    <property name="removeAbandoned" value="true" /> 
    <property name="removeAbandonedTimeout" value="${jdbc.removeAbandonedTimeout}" />

removeAbandoned在Spring中默认为false，即不移除遗弃的链接，这里我们设置为true，再设置超时时间为10,单位为秒，这样超过10s不进行新请求的链接将被释放回收，避免链接泄漏的情况发生。
目前的Spring 数据连接管理是自动建立和释放链接的，但是你需要使用jdbcTemplate或者使用SessionFactory.getCurrentSession()，其中，Session.getCurrentSession()是将Session绑定到Spring起的当前线程中，之后连接也就自然过度到Spring管理，自动释放；
但是如果使用SessionFactory.openSession()，是重新打开一链接，不与当前线程与事务绑定，这样如果你不手动close()的话，数据库连接就会泄漏。

Spring Session工厂
-------------------------------------
声明bean工厂：
<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
将数据源dataSource注入进入SessionFactory：
<property name="dataSource" ref="dataSource" />
命名策略：

<property name="namingStrategy">
    <bean class="org.hibernate.cfg.ImprovedNamingStrategy" />
</property>

<property name="hibernateProperties">
    <props>
        <prop key="hibernate.dialect">${hibernate.dialect}</prop>
        <prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
        <prop key="hibernate.format_sql">true</prop>
        <prop key="hibernate.generate_statistics">${hibernate.generate_statistics}</prop>
        <prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
    </props>
</property>


Transaction管理
-------------------------------------
事务管理也是Spring中的关键属性，首先声明事物：
<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
将SessionFactory注入到Transaction中：
<property name="sessionFactory" ref="sessionFactory"></property>
激活Spring注解方式：事务处理
<tx:annotation-driven transaction-manager="transactionManager" />

Bean注入
-------------------------------------
Bean注入是Spring特色之一，进行解耦，激活Spring注解方式：自动扫描，注入bean：
<context:component-scan base-package="com.neunn.cloud.base.*" />
这里是整个扫描一个包进行全初始化，通过Spring的注解@AutoWired直接使用

启动Spring对@AspectJ注解的支持:
<aop:aspectj-autoproxy/>
