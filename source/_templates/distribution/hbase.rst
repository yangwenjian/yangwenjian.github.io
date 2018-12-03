

=======================================
HBase
=======================================
* Linear and modular scalability.
* Strictly consistent reads and writes.
* Automatic and configurable sharding of tables
* Automatic failover support between RegionServers.
* Convenient base classes for backing Hadoop MapReduce jobs with Apache HBase tables.
* Easy to use Java API for client access.
* Block cache and Bloom Filters for real-time queries.
* Query predicate push down via server side Filters
* Thrift gateway and a REST-ful Web service that supports XML, Protobuf, and binary data encoding options
* Extensible jruby-based (JIRB) shell
* Support for exporting metrics via the Hadoop metrics subsystem to files or Ganglia; or via JMX

基本配置：

.. code:;

	hbase: 
	  zookeeper: 
		quorum: 10.9.49.17
		port: 2181
		
	<dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.3.0</version>
            <exclusions>
                <exclusion>
                    <groupId>jdk.tools</groupId>
                    <artifactId>jdk.tools</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-hadoop</artifactId>
            <version>2.5.0.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>javax.servlet</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>jdk.tools</groupId>
            <artifactId>jdk.tools</artifactId>
            <version>1.8</version>
            <scope>system</scope>
            <systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
        </dependency>

参考资料
=======================================
http://hbase.apache.org/book.html#quickstart
