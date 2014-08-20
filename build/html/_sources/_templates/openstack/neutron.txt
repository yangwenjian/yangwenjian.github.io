


======================================
Neutron
======================================
Neutron is the network controller in openstack.

Concept and Relationship
--------------------------------------
First, let me introduce some concepts I learned.

Network:    container of subnets, controll by tenant.

Subnet:     virtual network, including gateway, dhcp server, cidr, etc.

Port:       interface of subnet, router, or virutal machine.

Router:     Connection between subnets.

A network is ruled by tenant, and contains many subnets if needed.

