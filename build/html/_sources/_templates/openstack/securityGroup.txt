


===========================
SecurityGroup
===========================
Security Group（安全组）是虚拟机网络中的重要组成部分，可以用来控制虚拟机之间的网络流量。
反映到底层,就是利用iptables给宿主机添加iptables规则。

Openstack中nova有一个默认的安全组，如果不进行指定，nova就给寻及分配这个默认的安全组。
nova中有两个非常重要的组件，一个是db，一个是message queue，db用来记录各种状态，message queue用来在各个服务和各个节点之间传递消息，这种机制，在这里可以得到非常好的体现。

.. image:: ../../images/openstack/securitygroup1.png

API层进行rpc后，就交给compute.ComputeManager，进行依次调用。
ComputerManager中有一个driver，都知道这个driver默认的就是libvirt，还有就是init_host()方法比较重要，每个Manager都有这样一个方法，是相对应的服务在启动的时候调用的，用来进行一些初始化，主要做的工作是对网络进行初始化，建立一些初始的chain和rule。

.. image:: ../../images/openstack/securitygroup2.png

SecurityGroup属性
---------------------------------------------------------

每个安全组可以有多个规则，每个规则的属性如下：

ipProtocol: means the ip protocol of the security group, like ICMP, TCP, UDP, and so on.

fromport, toport: 这条安全组规则的影响端口范围，例如22就表示ssh，443表示http与https. To be noticed, fromport=-1 and toport=-1 means all port will be affated.

cidr: 无类别域间路由，这里区别于传统的A、B、C类网络地址，可以是任意进行IP类别划分, such as 192.168.250.222/28, it means that ip in that block will be affected by this security goup rule.

source group: it means this security group affact the members(virtual machine) of other security goups.

for example, we can add this command: nova secgroup-add-rule default tcp 22 22 172.31.0.224/28. Then we will get this:

+--------------+--------------+--------------+--------------+--------------+
| IP Protocol  | From Port    | To Port      | IP Range     | Source Group |
+--------------+--------------+--------------+--------------+--------------+
|     tcp      |    22        |     22       | 172.31.0.224 |              | 
+--------------+--------------+--------------+--------------+--------------+

目前Openstack社區將Neutron分離出來，Neutron的安全組更便於使用。
Nova的安全組默認只支持入口規則創建，而Neutron可以雙向控制。
