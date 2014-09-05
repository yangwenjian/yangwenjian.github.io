


==================================
Neutron
==================================
Neutron is the network controller in openstack.


Concpet and Relationship
----------------------------------
First, I will introduce some concept I have learned.

Network:    container of subnets.

Subnet:     virtual network including gateway, dns server, cidr.

Port:       virtual interface of router, subnet, virtual machines.

Router:     connector between subnets.

Adminstate: true if the administrator can control.

SegmentId:  use to add vlan tag or vxlan tag in network.

一个租户可以有多个Network，一个Network可以包含多个subnet，多个port，虚拟机通过port与subnet相连，subnet通过port和router进行相连，再连接到外部网络进行上网。

*一个network是一个vlan，带有一个vlan tag，这个vlan tage为segment id，一个租户可以有多个network，即可以有多个vlan，一个network可以有多个subnet，这些subnet属于同一个vlan。*



Neutron Network Type
-----------------------------------
There is several types in neutron network, 'FLAT', 'VLAN', 'GRE', 'VXLAN', 'LOCAL'.

Local is local network without any configuration.

Flat is flat network like all instances are in the same network plane.

Openstack允许可以多种网络类型共存，Vlan，Vxlan，GRE可以共存。

