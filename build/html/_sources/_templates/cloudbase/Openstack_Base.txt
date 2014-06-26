


=======================================
OpenStack Base 开发总结
=======================================
东网云平台的架构和设计



Base层的作用
=======================================



Base层的用法
=======================================




计费模块
---------------------------------------
计费功能在模板中主要实现，虚拟机，网络，块存储单独计费，其中虚拟机计费是首要需求，虚拟机计费可以根据模板费用×晕新时间进行计费。
首先实现整个的费用平台，加入充值接口，实现扣费模块，固定套餐计费等组件。

首先计算模板的费用，每个模板含有的cpu数目，内存大小，网络带宽作为模板费用的
之后可以进行虚拟机运行时间的统计，可以利用openstack自带的组件ceilometer或者使用nova进行虚拟机运行时间的统计。
如果能有信息实时推送到base层，这样就可以实现实时计费，不能采取base轮询的策略，这样比较耗费资源。最好是nova或者ceilometer或者我们自己的监控平台进行推送到base层实现实时计费。
如果调用ceilometer查询主机的运行时间，只能实现事后计费，固定实际（进行计费统计并扣费）。
autocaling的计费和负载均衡的计费暂时先不考虑。

快照也需要收钱，按照快照的次数和快照大小进行计费。

网络的计费模式主要取决于路由器，按照路由器的个数进行计费（青云）。neutron是否有配额功能？

块存储也有相应的模板进行计费，高性能的存储和地性能的存储分开计费，按照大小和使用时间进行计费，暂时不实现回收站机制。

消费记录也是用户的需求点之一，我们从ceilometer中获取log进行解析后存储在base层，然后发送给用户。

.. image:: ../../images/cloud_billing_mapreduce.jpg

整体的架构如下图所示：

.. image:: ../../images/cloud_billing_arcitecher.jpg


JBilling
=======================================
JBilling is an enterprise billing system base on spring, web server.
It has four edition: Community, Enterprise, Telco, Telco Hosted. Each after is more functional.

JBilling is a seperated system of billing which contains functions like: Core, Accounts, Invoices, Payments, Intergrations, Product Management, Mediation, Support, Deployment.


so yet, I can not find a way to intergrate JBilling in our system. But JBilling(Community) is an opensource project, we can add some of its componets.

