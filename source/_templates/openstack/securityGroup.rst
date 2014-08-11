


===========================
SecurityGroup
===========================
Security Group（安全组）是虚拟机网络中的重要组成部分，可以用来控制虚拟机之间的网络访问。


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

