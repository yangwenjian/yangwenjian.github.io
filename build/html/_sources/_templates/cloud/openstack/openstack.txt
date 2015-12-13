


================================
OpenStack Learning
================================
How I learn about OpenStack!

Introduction
================================
1. Compute（Nova）提供计算虚拟化服务，是 OpenStack 的核心，负责管理和创建虚拟机。它被设计成方便扩展，支持多种虚拟化技术，并且可以部署在标准硬件上；
#. Object Storage（Swift）提供对象存储服务，是一个分布式，可扩展，多副本的存储系统；
#. Block Storage（Cinder），提供块存储服务，为 OpenStack 的虚拟机提供持久的块级存储设备。支持多种存储后端，包括 Ceph，EMC等；
#. Networking（Neutron）提供网络虚拟化服务，是一个可拔插，可扩展，API 驱动的服务；
#. Dashboard 提供了一个图形控制台服务，让用户方便地访问，使用和维护 OpenStack 中的资源；
#. Image（glance）提供镜像服务，它旨在发现，注册和交付虚拟机磁盘和镜像，支持多种后端；
#. Telemetry（Ceilometer）提供用量统计服务，通过它可以方便地实现 OpenStack 计费功能；
#. Orchestration（Heat）整合了 OpenStack 中的众多组件，类似 AWS 的 CloudFormation，让用户能够通过模板来管理资源；
#. Database（Trove）基于 OpenStack 构建的 database-as-a-service。
#. ImageMarket（Murano）OpenStack镜像市场。
#. HealthCheck（OSTF）OpenStack健康检查。


Learning Path
================================
Use OpenStack to Build our own Cloud System
1.Build base layer to merge other cloud kernel such as CloudStack, Vmare;

    1.1 Use jclouds to be our OpenStack SDK;
    
    1.2 Build our own beans correspondence jclouds beans;

2.Build rest api service with jersy;

3.Build monitor system named argus based on zabbix;

4.Build our own storage system based on ceph;

5.Build management and operation service base on base layer;

OSLO
==============================
在Oslo项目出现之前，Openstack的每个组件都要自己实现处理配置文件的代码，这样就导致了一系列重复代码的产生，这是程序员所不能忍受的，因此Oslo就解决这类问题。

Oslo需要解决的几点问题：

1) command line option parsing;
2) common command line options;
3) configuration file parsing;
4) option value lookup.

Oslo.conf在使用时，需要先声明配置项的名称、定义类型、帮助文字、缺省值等，然后再按照事先声明的配置项，对CLI或conf中的内容进行解析。

::

    common_opts = [
        cfg.StrOpt('bind_host',
            default='0.0.0.0',
            help='IP address to listen on'),
        cfg.IntOpt('bind_port',
            default=9292,
            help='Port number to listen on')
    ]

OpensStack from Netease
================================
网易利用OpenStack部署自己的私有云服务，为游戏和互联网应用提供基础平台。

提高了机器的利用率，减少了运维开销和人员，提高了应对突发业务的能力。

参考资料：http://www.ibm.com/developerworks/cn/cloud/library/1408_zhangxl_openstack/#resources.

网易部署方案
--------------------------------
总结下网易在部署中的一些技巧。

1. keystone后端使用mysql，使用memcache存放token，所有服务（nova，glance，neutron）的keystoneclient都使用memcache作为token的缓存；
2. 多region部署方案可以使得整个openstack资源池既可以支持nova-network和neutron两种网络部署方案，通过共享一个keystone实现用户单点登录；

.. image:: images/openstack_multiregion.jpg

3. 为提高硬件利用率，将服务器分为计算节点，控制计算节点；
4. 对外提供 API 的服务有 nova-api-os-compute，nova-novncproxy ,glance-api，keystone。这类服务的特点是无状态，可以方便地横向扩展，故此类服务均部署在负载均衡 HAProxy 之后，并且使用 Keepalived 做高可用；

.. image:: images/openstack_node.jpg

5. 为了保证服务质量和便于维护，我们没有使用 nova-api，而是分为 nova-api-os-compute 和 nova-api-metadata 分别管理。外部依赖方面，网易私有云部署了高可用 RabbitMQ 集群和主备 MySQL，以及 memcache 集群；
6. 网络规划方面，网易私有云主要使用 nova-network 的 FlatDHCPManager+multi-host 网络模式，并划分了多个 Vlan，分别用于虚拟机 fixed-ip 网络、内网浮动 IP 网络、外网网络；
7. 运维使用类似nagios的自主研发系统，部署使用puppet，bug定位使用Stack Tach；

优化方案
--------------------------------
网易选用kvm做底层虚拟化，使用libvirt做计算驱动，linux发行版选择Debian。
选择了Debian社区的Linux 3.10.40内核源代码，并且打开了CPU/mem/blkio等cgroup配置选项以及user namespace等namespace选项，自己编译了一个适配网易私有云的Linux内核。

**cpu优化**

2 个 VCPU 分别绑定到不同 numa 节点的非超线程核上和分配到一对相邻的超线程核上的性能相差有 30%~40%（通过 SPEC CPU2006 工具测试）。
由于cpu0用于处理中断请求，我们将cpu0～3预留出来，libvirt的配置如下：

::

    <vcpu placement='static' cpuset='4-23'>1</vcpu>
    <cputune>
        <shares>1024</shares>
        <period>100000</period>
        <quota>57499</quota>
    </cputune>

**内存优化**

内存方面采取关闭KVM内存共享，打开透明大页。

::

    echo 0 > /sys/kernel/mm/ksm/pages_shared
    echo 0 > /sys/kernel/mm/ksm/pages_sharing
    echo always > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
    echo 0 > /sys/kernel/mm/transparent_hugepage/khugepaged/defrag

这些配置对云主机的cpu性能有大概7%的提升（资料来自IBM developer works）。


**磁盘优化**

KVM的disk cache: none cache方式；

disk io scheduler: cfq；

IO QOS: Nova 执行命令方式写入到 cgroup 中；

network IO: 开启了 vhost_net 模式。

经验分享
---------------------------------
建议跟进社区版本，与社区保持同步；

不要闭门造车，多参考社区方案；

升级问题，避免停止服务；

配置项与硬件有很大关系，测试通过才能上线；

避免配额之和超出容量，会有各种问题；

网络规划提前做好；

网络安全和信息安全需要引起重视。


