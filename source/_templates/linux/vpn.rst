


==============================
VPN
==============================
VPN: virtual private network

xl2tpd客户端

::

 1.首先安装xl2tp，zypper in xl2tpd ppp (ubuntu和centos使用相应工具)安装。
 2.配置/etc/ppp/peers/testvpn.l2tpd，修改如下两个配置：
 user "dev"
 password "Neunn@)!$"
 3.配置/etc/xl2tpd/xl2tpd.conf
 [lac client]
 name = ops                                  
 lns =61.161.217.98                                   
 pppoptfile = /etc/ppp/peers/testvpn.l2tpd         
 ppp debug = no
 4.启动xl2tp
 /etc/init.d/xl2tpd start
 echo 'c client' > /var/run/xl2tpd/l2tp-control
 5.添加相应路由（此处酌情考虑）
 6.断开vpn
 echo 'd client' > /var/run/xl2tpd/l2tp-control
 /etc/init.d/xl2tpd stop
