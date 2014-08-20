


====================================
Linux Network Tool
====================================
Linux has planty of tools of network, I will introduce some of them which are used in my develop work.

Curl
====================================
Curl stands for command line url viewer.

Example:

发送post请求获取openstack的token

::

    curl -X POST  -i -H "Content-Type:application/json" -d '{ "auth" : { "tenantName" : "admin","passwordCredentials" : { "username" : "admin","password" : "admin"}}}' http://192.168.250.222:5000/v2.0/tokens

发送get请求images信息列表

::

    curl -X GET -H "Content-Type:application/json" -H "X-Auth-Token:..." http://192.168.200.7:9292/v1/images/detail
