


=====================================================
OpenStack Base Back Stage NeunnManager
=====================================================
我们将设计一个后台管理系统，主要职责是Tenant（租户）、User（用户），Host（主机）、HostType（主机类型）、Availability Zone（可用域、以下简称AZ）、Quato（配额）的管理维护，计费相关功能。

与Base层的区别有两点：

1. 权限不同。这里所有的对于Openstack的操作（租户、用户、AZ、配额）都使用Admin权限，而这个权限base层对用户是禁止的（安全原因）。
2. 加入了自维护的数据库，增强OpenStack对与宿主机（Host）的管理，加入主机类型，主机与主机类型对应关系（Host表中加入一个字段），主机与AZ的对应关系（新建一张映射表）。

这里，我的观点略有不同，主机与AZ的映射关系OpenStack已经有，只是信息很少，关于主机部分只有HostName（唯一）。
这里我们可以同步维护一张表，但我认为只要在Host表中加一个字段AZId即可（因为AZ与Host是1：n的关系），没有必要新建一张表，
其实关于这里的对象转换我也不是很理解，要知道对象转换非常吃资源，新建对象，get/set的转换等，base层的风格延续了。
这里暂时放下个人观点，继续介绍我们团队的设计。

Host与AZ对应关系的维护
----------------------------------------------------
由于设计的问题，我们要同时维护自己的Host表，Host与AZ的对应关系表，并维护OpenStack的Host与AZ(Host Aggregate)，这里就要注意与OpenStack的同步，避免信息混乱。

AZ中Host列表的更新
````````````````````````````````````````````````````
根据产品经理的要求，我们做类似与OpenStack风格的更新AZ，传入的参数是AZId，更新后的Host列表，这里我的思路如下：

1. 查找AZ中已有的Host列表，与传入的Host列表进行集合的加减法，得到两个列表，分别是待删除列表与待添加列表；
2. 轮循待删除列表，对于里面的每台Host进行如下事物操作:

    2.1. 利用JClouds将主机从HostAggregate中移除；
    
    2.2. 删除AZ-Host表中对应的列；

    2.3. 将Host表中对应的列中AvailabilityZone字段置为Null，将Status置为free；
3. 轮循待添加列表，对于里面的每台Host进行如下事物操作：

    3.1. 利用JClouds将主机添加到相应的HostAggregate中；
    
    3.2. 添加AZ-Host表中对应的项；
    
    3.3. 将Host表中对应的列中AvailabilityZone字段置为该AZ，将Status置为busy；

Host,AvailabilityZone与OpenStack的同步问题
---------------------------------------------------
NeunnManager维护的Host,AZ信息必须与OpenStack保持同步，并且以OpenStack为准，否则就会出现AZ与Host的管理混乱。

Host--计算节点
```````````````````````````````````````````````````
计算节点，也就是Openstack中的host，与AvailabilityZone是n:1的关系，并且带有一个服务器类型，这个类型是服务器的配置，显示的Host带有的虚拟CPU，虚拟RAM，虚拟VOLUME的大小。

因为我们要实现物理隔离，就必须将AvailabilityZone私有化，让其属于某个tenant，这样用户在创建实例的时候只能选择自己的AZ，不能在别人的AZ上创建实例。

1. 计算节点的字段为：id, insertTime, lastUpdateTime, **name**, **availabilityzone**, status, typeid, typename, **ip**, vCpus, vRams, vVolumes.
2. AvailabilityZone的字段为：id, insertTime, lastUpdateTime, **azName**, tenantId, tenantName, status, region, **aggregateId**.
3. HostAzmapping的字段为：id, insertTime, lastUpdateTime, hostId, azId.

这里，黑色字体表示从OpenStack处获得，必须与OpenStack保持同步。这里最重要的关系维护在HostAzMapping这张表中，但是由于使用的是hostId和azId，不能直接与OpenStack进行同步，因为OpenStack是通过名称（唯一）进行关系维护的。
