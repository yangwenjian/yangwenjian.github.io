
===========================================
Varnish
===========================================

Varnish是什么
===========================================
Varnish是一款高性能的开源HTTP加速器，也就是我们常用的URL缓存。
Varnish支持使用内存、临时文件、永久性文件进行缓存，当然，如果内存够用的话，使用内存能达到最好的效果，在我们的配置中，使用内存作为存储空间。

Varnish的原理
===========================================
请参考Varnish的流程图：


如何配置Varnish
===========================================

配置文件示例如下：

..code::

	backend tomcat1 {
		.host = "10.10.18.65";
		.port = "8080";
		.probe = {
			.url = "/storefiles/";
			.interval = 5s;
			.timeout = 1s;
			.window = 5;
			.threshold = 3;
		}
	}
	
	director tomcat random {
		{.backend = tomcat1; .weight = 5;}
		{.backend = tomcat2; .weight = 5;}
		{.backend = tomcat3; .weight = 5;}
	}
	sub vcl_recv {
		if (req.url ~ "\.(css|js|html|htm|gif|jpg|jpeg|png|bmp|tiff|tif|ico|img|mp3|ogg|swf|flv)($|\?)") {
			remove req.http.cookie;
			return (lookup);
		} else {
			return (pass);
		}
	}
	sub vcl_fetch {
		if (beresp.status == 500 || beresp.status == 501 || beresp.status == 502 ||
			beresp.status == 503 || beresp.status == 504 || beresp.status == 500) {
			return (restart);
		}
	}
	sub vcl_deliver {
		if (obj.hits > 0) {
			set resp.http.X-Cache = "HIT from www.xikang.com";
		} else {
			set resp.http.X-Cache = "MISS from www.xikang.com";
		}
		return (deliver);
	}
	
Varnish的基本命令：
-------------------------------------------
Varnish启动：
varnishd -f /home/query/vcl.conf -s malloc,1G -T 127.0.0.1:2000 -a 0.0.0.0:7021 -d
之后进行启动child，键入start命令。
通过Varnishlog可以查看详细的log信息。

这里-f指配置文件位置，-s指存储方式和大小，-T是管理端口指定，-a是代理端口指定，-d是按照deamon程序运行。

Varnish最佳实践
===========================================
Varnish是利用反向代理进行缓存的，这个我开始的时候配置了好久，以为是一种拦截器装置，可以截获某个端口的数据，进行缓存和转发。
使用反向代理，更简单易用。

1.是否可以用cookie+url做索引去缓存页面？
varnish的hash值可以任意修改，如sub vcl_hash {hash_data(req.http.X-Country-Code);}，将任意的标准放入hash中。
这里我将hash_data(request.http.cookie);加入到vcl_hash中，经不通浏览器，不同机器测试，确实将cookie值放入到的了hash中。

2.缓存在Varnish中的是如何存储的？
目前不支持，只能看到一些状态参数，内容太多，显示出来很容易造成varnish停止服务。
但我们可以使用varnishstat查看一些基本参数。

3.url是否支持正则表达式，如何设计我们的url patter？
Varnish支持正则表达式或者ACL列表，如果使用ACL列表，需要向backend一样在前面声明。
在我们的设计中，我建议使用前缀来区别是否走缓存。例如，我们使用/portal前缀来标识其为公共浏览页面；也可以使用后缀来表示某些请求；也可以使用reqeust的类型来决定是否走缓存；这些都标识在vcl_recv方法中。
同时，在vcl_fetch函数中，我们也可以使用response的一些属性来决定是否缓存。
如何特殊的url，可以通过设置request或者response的Cache-Control来进行控制。

.. code::

	sub vcl_fetch {
		if (beresp.status == 500 || beresp.status == 501 || beresp.status == 502 ||
			beresp.status == 503 || beresp.status == 504 || beresp.status == 500) {
			return (restart);
		}
		if (beresp.status == 401 || beresp.status == 403 || beresp.status == 404) {
			return (hit_for_pass);
		}
		#WEB服务器指明不缓存的内容，varnish服务器不缓存
		if (beresp.http.Pragma ~ "no-cache" || beresp.http.Cache-Control ~ "no-cache" ||
			beresp.http.Cache-Control ~ "private") {
			return (hit_for_pass);
		}	
		if (req.request == "GET" && req.url ~ "\.(css|js|html|htm)$") {
			set beresp.ttl = 300s;
		}
		if (req.request == "GET" && req.url ~ "\.(gif|jpg|jpeg|png|bmp|tiff|tif|ico|img)$") {
			set beresp.ttl = 3600s;
		}
		if (req.request == "GET" && req.url ~ "\.(mp3|ogg|swf|flv)$") {
			set beresp.ttl = 10d;
		}		
		return (deliver);
	}


4.缓存的时间如何设置？
修改beresp.ttl变量如：set beresp.ttl = 1200s。
同时，Varnish也接受过期对象的时间设置。

5.什么样的请求不能走缓存？
一般的post请求，带有动作的请求，实时性请求，建议直接走pass函数，直接从后台取，不进行缓存。

参考资料
===========================================
https://www.varnish-cache.org/docs/4.0/index.html
