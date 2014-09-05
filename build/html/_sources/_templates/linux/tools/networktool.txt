


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

    curl -X GET -H "Content-Type:application/json" -H "X-Auth-Token:..." http://192.168.250.222:9696/v2.0/subnets

curl 对于http请求默认的方法是get，利用-X可以显示的指定相应方法，像上述两个例子可以省略-X参数，第一个例子中，加上-d表示为post。

下载图片

::

    curl -o 1.jpg http://cgi2.tky.3web.ne.jp/~zzh/screen1.JPG
    curl -O http://cgi2.tky.3web.ne.jp/~zzh/screen[1-10].JPG
    curl -o #2_#1.jpg http://cgi2.tky.3web.ne.jp/~{zzh,nick}/[001-201].JPG



Ping
=====================================
Ping is the most usual tool we use in network.
Ping is base on icmp protocol.

::

 Knight-Linux:/home/knight # ping 192.168.250.3
 PING 192.168.250.3 (192.168.250.3) 56(84) bytes of data.
 64 bytes from 192.168.250.3: icmp_seq=1 ttl=62 time=0.290 ms
 64 bytes from 192.168.250.3: icmp_seq=2 ttl=62 time=0.384 ms
 64 bytes from 192.168.250.3: icmp_seq=3 ttl=62 time=0.387 ms
 ^C
 --- 192.168.250.3 ping statistics ---
 3 packets transmitted, 3 received, 0% packet loss, time 2000ms

1) 64bytes 是icmp封包大小的预设值，可以用-s 200等来代替；
2) icmp_seq=1 意识是侦测次数；
3) ttl=62 每经过一个带有mac的节点，如router，bridge等，ttl就减少1,预设值为255,可以用-t 200等来设置；
4) time=0.387ms 回应时间，越小说明网络环境越好。

Tcpdump
==========================================
Tcpdump能截获当前所有通过本机网卡的数据包，有着灵活的过滤机制。由于只能截获本机网卡数据，只能用于自检或者在网关的服务器上。

例如：

::

 tcpdump -i eth0 src host 192.168.0.5
 tcpdump -i eth0 src host 192.168.0.5 and dst port 80

在本机上使用 *curl -i www.sina.com* 访问新浪，监听得到如下结果：

::

 root@192-168-250-222:~# tcpdump -i eth0 dst port 80
 tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
 listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
 16:16:45.075410 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [S], seq 899886478, win 14600, options [mss 1460,sackOK,TS val 1276140011 ecr 0,nop,wscale 7], length 0
 16:16:45.247567 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 4291915039, win 14600, options [nop,nop,TS val 1276140054 ecr 4153792769], length 0
 16:16:45.247686 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [P.], seq 0:76, ack 1, win 14600, options [nop,nop,TS val 1276140054 ecr 4153792769], length 76
 16:16:45.421770 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 327, win 15544, options [nop,nop,TS val 1276140097 ecr 4153792943], length 0
 16:16:45.421810 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 1775, win 17376, options [nop,nop,TS val 1276140097 ecr 4153792943], length 0
 16:16:45.421960 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 3223, win 20272, options [nop,nop,TS val 1276140097 ecr 4153792943], length 0
 16:16:45.594158 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 4671, win 23168, options [nop,nop,TS val 1276140140 ecr 4153793115], length 0
 16:16:45.594255 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 6119, win 26064, options [nop,nop,TS val 1276140140 ecr 4153793115], length 0
 16:16:45.594344 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 7567, win 28960, options [nop,nop,TS val 1276140140 ecr 4153793115], length 0
 16:16:45.594454 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 9015, win 31856, options [nop,nop,TS val 1276140140 ecr 4153793116], length 0
 16:16:45.594607 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 10463, win 34752, options [nop,nop,TS val 1276140140 ecr 4153793116], length 0
 16:16:45.766313 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 11911, win 37648, options [nop,nop,TS val 1276140183 ecr 4153793288], length 0
 16:16:45.766477 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 13359, win 40544, options [nop,nop,TS val 1276140183 ecr 4153793288], length 0
 16:16:45.766567 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 14807, win 41992, options [nop,nop,TS val 1276140183 ecr 4153793288], length 0
 16:16:45.766718 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 16255, win 41992, options [nop,nop,TS val 1276140184 ecr 4153793288], length 0
 16:16:45.766855 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 17703, win 41992, options [nop,nop,TS val 1276140184 ecr 4153793288], length 0
 16:16:45.766964 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 19151, win 41992, options [nop,nop,TS val 1276140184 ecr 4153793288], length 0
 16:16:45.767176 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 20172, win 41992, options [nop,nop,TS val 1276140184 ecr 4153793288], length 0
 16:16:45.767290 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [F.], seq 76, ack 20172, win 41992, options [nop,nop,TS val 1276140184 ecr 4153793288], length 0
 16:16:45.939178 IP 192.168.250.222.39382 > 12.130.132.30.http: Flags [.], ack 20173, win 41992, options [nop,nop,TS val 1276140227 ecr 4153793461], length 0

