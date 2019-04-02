

=================================================================
Elastic Logstash Kibana
=================================================================

Introduction
=================================================================

Elastic
-----------------------------------------------------------------
Features：

* Real-Time Data
* Real-Time Advanced Analytics
* Massively Distributed
* High Availability
* Multitenancy
* Full-Text Search
* Document-Oriented
* Schema-Free
* Developer-Friendly, RESTful API
* Per-Operation Persistence
* Apache 2 Open Source License
* Build on Apache Lucene

Installation
=================================================================
Download and start

.. code::

    curl -L -O https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.1.1/elasticsearch-2.1.1.tar.gz
    tar -xvf elasticsearch-2.1.1.tar.gz
    cd elasticsearch-2.1.1/bin
    ./elasticsearch

We can check the cluster's health.

.. code::

    curl 'localhost:9200/_cat/health?v'

We can CRUD the indices. Since there is only one node in our cluster, the health would be yellow, because elastic can not write indices replicas.

.. code::

    curl 'localhost:9200/_cat/indices?v'
    curl -XPUT 'localhost:9200/customer?pretty'

Logstash
-----------------------------------------------------------------
Download and start

.. code:: 

    curl -L -O https://download.elastic.co/logstash/logstash/logstash-2.1.1.tar.gz
    tar -xvf logstash-2.1.1.tar.gz
    
Prepare and edit logstash.conf.

.. code::

    input{
	    file{
		    path=>"/home/knight/software/enviroment/apache-tomcat-8.0.30/logs/catalina.out"
	    }
	    file{
		    path=>"/home/knight/downloads/test.log"
	    }
    }

    filter {
	    grok {
		    match => { "message" => "%{COMBINEDAPACHELOG}" }
	    }
	    geoip {
		    source => "clientip"
	    }
    }

    output{
	    elasticsearch {
		    hosts => ["localhost:9200"]
	    }
	    stdout{
	    }
    }

Start logstash. 

.. code::

    ./logstash agent -f logstash.conf

Start Kibana.

.. code::

    ./bin/Kibana

Create a index in elasticsearch.

    


    
References
=================================================================
https://www.elastic.co/products/elasticsearch
