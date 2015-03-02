


=====================================
Docker
=====================================
Docker is a virtualization tool Based on Linux Container(LXC).
Useful and easy-to-use.

Introduction
=====================================
Docker is application container, to some extent, like virtual machines.
But there are a lot of differences:

* Virtualization is based on physical cpu and memory, while docker is based on OS, using containerization technique, even running on virtual machine.
* Virtualization refers to system image, called 'system', while docker is called 'container', which is more suitable for light application, such as redis, or memcached.
* Virtualization uses snapshot to save the state. Docker is lower cost and protable. Docker imports code version control, which is more easy to switch between versions.
* Virtualization is more complex at building system. Docker uses dockerfile to build containers, which is faster. Dockerfile can access on the host, which is more convenient.


Docker's features:

* File system isolation.
* Resource isolation, like cpu and memory.
* Network isolation: every container has its own network interface and ip.
* Writing copy: makes deploy faster, save more disk space and memory space.
* Log system.
* Change management.
* Interactive shell.

Docker Design
===================================
Docker uses a client-server architecture.
The docker client and daemon communicate via sockets or through restful api.

The Docker Daemon
-----------------------------------
The docker daemon runs on a host machine. The user does not directly interact with the daemon, but instead through the Docker client.

The Docker client
-----------------------------------
The Docker client, in the form of the docker binary, is the primary user interface to Docker. 
It accepts commands from the user and communicates back and forth with a Docker daemon.

Inside Docker
-----------------------------------
Docker contains these parts:

1) Docker images. Docker images are the build components of docker. It can easily created, downloaded, updated.
2) Docker registries. Docker registries hold images, like public or private stores.
3) Docker containers. Docker containers are the run compontes of docker, like directory, but are isolated and secure application platform.

Docker Images
```````````````````````````````````
Each docker images consists of a series of layes.
Docker makes use of union file system to combine these layers into a single image.
Union file systems allow files and directories of separate file system, known as branches, to be transparently overlaid, forming a single coherent file system.

One of the reasons docker is so lightweight is because of these layers. When we change a docker image--for example, update an application to a new version--a new layer was built. 
With only that layer built, docker is faster than virtual machines.

Docker Registries
```````````````````````````````````
Docker Hub provides both public and private storage for images. Public storage is searchable and can be downloaded by anyone. Private storage is excluded from search results and only you and your users can pull images down and use them to build containers.

Docker Containers
```````````````````````````````````
A container consists of an operating system, user-added files, and meta-data.
When we run *docker run -i -t ubuntu /bin/bash* , docker does the following:

::

    1) pulls the ubuntu image;
    2) create a new container;
    3) Allocate a file system and mounts a read-write layer;
    4) Allocate a network/bridge interface, allowing docker containers talk to the host;
    5) Sets up an IP address;
    6) Excutes a process that you specify;
    7) Captures and provides application output.

The Underlying Technology
====================================
Docker is written in Go and makes use of serveral Linux kernel features to deliver the functionality we have seen.

Namespace
------------------------------------
Docker use namespace to isolate the workspace.

Some of the namespaces that docker uses are:

::

    1) The pid namespace: used for process isolation;
    2) The net namespace: used for managing network interfaces;
    3) The ipc namespace: used for managing access to IPC resources;
    4) The mnt namespace: used for managing mounting point;
    5) The uts namespace: used for isolating kernel and version identifiers.

Control Groups
-------------------------------------
Control groups allow Docker to share available hardware resources to containers and, if required, set up limits and constraints. 
For example, limiting the memory available to a specific container.

Union File System
-------------------------------------
Docker can make use of several union file system variants including: AUFS, btrfs, vfs, and DeviceMapper.

这里关于AUFS和DeviceMapper简单介绍一下：

1) AUFS是一种Union FS，简单来说是将不同的目录挂载在一个虚拟文件系统下，优点是支持容器间共享可执行及可共享的运行库。在ubuntu中，docker就使用aufs driver，我们可以通过目录来看到docker容器里的文件。
2) DeviceMappper是一种逻辑设备到物理设备的映射框架机制（注意，这里是机制，而策略是用户层的概念，linux主张策略与机制分开）。我们熟悉的LVM就是在DeviceMapper框架下运行的。在opensuse中，docker就使用DeviceMapper driver，我们可以看见一个大块的文件（100G），每个容器在10G之内（关于DeviceMapper的介绍在Linux LVM部分）。


Container Format
-------------------------------------
Docker combines these components into a wrapper we call a container format. The default container format is called libcontainer.
Docker also supports traditional Linux containers using LXC. 
In the future, Docker may support other container formats, for example, by integrating with BSD Jails or Solaris Zones

Exersice
=====================================
今天将base层接口迁移到新的更大的openstack环境中，这个环境下，所有的endpoint都是用主机名表示的，这样的好处是便于维护和区分各个主机的作用。

但引发一个问题，base层是部在dokcer容器中，docker不支持/etc/hosts主机名解析，这个文件根本就是readonly的。于是我请教我的同事。
可以在docker启动的时候加入参数

::

    docker start DOCKER_ID -v /etc/hosts:/etc/hosts:ro

今天（2014.09.17）Docker1.2版本发布，支持/etc/hosts文件解析主机名IP，这正好满足了我今天的需求。
由于ubuntu官方的源daocker不是最新版的，只能将docker官方源加入到源中：

::

    echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list

重新安装后，docker进程重新启动，所有的容器都停止了（之前应该做些备份处理的）。

这里有个小插曲：

Docker每次重启的时候都会DHCP一个新的IP，这次升级后它的ssh私钥发生了变化，原来的免密码登陆失效了，而且直接报错。
这里是这样的，ssh在连接的时候将server端的public key保存到本地的~/.ssh/know_hosts文件中，只要删除这个文件中的相应内容，就可以重新密码连接了。

其实完全可以用其他工具进行连接容器，这里推荐使用nsenter，轻量级连接docker工具，简单易用。
安装（这里暂不推荐最新版2.25,编译的时候有问题，没解决）：

::

    curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-
    cd ../util-linux-2.24/
    ./configure --without-ncurses
    make nsenter
    cp nsenter /usr/local/bin
    docker ps -a
    PID=$(docker inspect --format '{{.State.Pid}}' bfcd9910faee)
    nsenter --target $PID --mount --uts --ipc --net --pid

之后就跟ssh上去一样，可以操作容器了。

Docker实践中遇到的问题
-----------------------------------
今天base和portal第一个版本发布，我将部署docker容器作为发布的运行容器。
第一次写dockerfile，参考了同事的资料：

::
    
    FROM ubuntu
    MAINTAINER yangwenjian <yangwj@neunn.com>

    RUN apt-get update 
    ADD tomcat7 /usr/local/
    ADD jdk1.7.0_55 /usr/lib/
    ADD profile /etc/
    EXPOSE 8888 22

build后产生新的镜像，结果怎么run这个镜像也跑不起来，直接镜像就退出，通过docker logs也看不出什么。

运行我镜像列表的里的所有镜像，发现都是同一个毛病，求助于同事，同事查看了一通后也没发现明显的问题。他只是觉得镜像有问题，最后发现是我在构建的时候下载镜像的过程中断网了，结果镜像没有下去，有问题，当时就被公司的网络耍了一把。

用了新的镜像后发现docker file有些内容没有写进去，profile是写进去了，但是tomcat和jdk都没有进入文件系统中，其实是我的dockerfile写法有问题，ADD添加文件夹的时候和我们观念上的copy文件夹不同，需要给文件夹指定名称。正确的写法如下：

::

    #his is a docker file to create container for base/portal deployment
    FROM ubuntu:neunn
    MAINTAINER yangwenjian <yangwj@neunn.com>

    RUN apt-get update 
    ADD tomcat7/ /usr/local/tomcat7
    ADD jdk1.7.0_55/ /usr/lib/jdk1.7.0_55
    ADD profile /etc/
    EXPOSE 8888 22 

这里启动后会自动加载/etc/profile文件，就想linux系统启动一样。

某天突然停电，重新启动服务器后，再启动所有docker容器，发现base层服务出现连接超时！
原因是docker容器再重新启动后会覆写/etc/hosts文件，之前加的host与IP的对应表都消失了！
这是docker的一种特性吧，这里推荐在启动时加入-v挂载本地文件到docker容器中，这样就会永久生效。

Docker中的进程
````````````````````````````````````
Docker虽然将各个容器进行隔离，但是在宿主机中依然能观测到docker中的各种进程。

某天我在调物理服务器的数据库，因为我shutdown mysql后发现还有mysql进程，我当时以为没有正常关闭就kill掉了（事后才知道是某个docker中的mysql进程）；
第二天测试工程师来找我问我有没有动过他的数据库，我说我调的物理服务器的数据库，并没有动你docker内部的数据库，我进去调试发现mysql进程根本没启动，我就说你这进程都没了，肯定不好使啊。

后来我突然意识到可能是当天的一个kill动作产生的结果，就在物理服务器中查看mysql进程，果然有两个：

::

    root      7647  0.0  0.0   4444   752 ?        S    Feb26   0:00 /bin/sh /usr/bin/mysqld_safe
    mysql     8059  1.8  0.2 13835468 377860 ?     Sl   Feb26 100:52 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --user=mysql --log-error=/var/log/mysql/error.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306
    root      8294  0.0  0.0   4444   752 ?        S    03:11   0:00 /bin/sh /usr/bin/mysqld_safe
    landsca+  8651  0.2  0.0 689568 60520 ?        Sl   03:11   0:00 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --user=mysql --log-error=/var/log/mysql/error.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306
    mysql     9310  0.1  0.0 381052 33496 ?        Ssl  03:14   0:00 /usr/sbin/mysqld

我不死心，又去看ssh进程，这下就都明白了：

::

    root      1992  0.0  0.0  61364  2280 ?        Ss    2014   1:41 /usr/sbin/sshd -D
    root      2789  0.0  0.0  61364  1176 ?        Ss   Feb12   0:00 /usr/sbin/sshd
    root      4340  0.0  0.0  61364  1068 ?        Ss   Feb12   0:00 /usr/sbin/sshd
    root      4753  0.0  0.0  61364  1072 ?        Ss   Feb12   0:00 /usr/sbin/sshd
    root      7733  0.0  0.0  61364  1300 ?        Ss   Feb12   0:00 /usr/sbin/sshd
    root      9885  0.0  0.0 105628  4316 ?        Ss   03:15   0:00 sshd: root@pts/5    
    root     10152  0.0  0.0  10468   916 pts/5    S+   03:15   0:00 grep --color=auto ssh
    root     13235  0.0  0.0  61364  1652 ?        S    Feb10   0:00 /usr/sbin/sshd -D
    root     16146  0.0  0.0  61364  1636 ?        S    Feb10   0:00 /usr/sbin/sshd -D
    root     16638  0.0  0.0  61364  1636 ?        S    Feb10   0:00 /usr/sbin/sshd -D
    sshd     26085  0.6  0.0 551020 46384 ?        Sl   Feb10 186:52 /usr/bin/mongod --unixSocketPrefix=/var/run/mongodb --config /etc/mongodb.conf run
    root     31001  0.0  0.0  61364  1148 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     31064  0.0  0.0  61364  1284 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     31134  0.0  0.0  61364  1288 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     31225  0.0  0.0  61364  1144 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     41461  0.0  0.0  61364  1148 ?        Ss   Feb09   0:00 /usr/sbin/sshd -D
    root     43786  0.0  0.0 106856  5548 ?        Ss   01:52   0:02 sshd: root@pts/12   
    root     43904  0.0  0.0  13040  1192 ?        Ss   01:52   0:00 /usr/lib/openssh/sftp-server
    root     44953  0.0  0.0  44140  2956 pts/12   S+   01:55   0:00 ssh root@172.17.0.20
    root     44954  0.0  0.0  63436  3520 ?        Ss   01:55   0:00 sshd: root@pts/0    
    root     45353  0.0  0.0  61364  1680 ?        Ss   Feb11   0:00 /usr/sbin/sshd -D
    root     46987  0.0  0.0  61364  1144 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     51025  0.0  0.0  61364  1152 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     51678  0.0  0.0  61364  1140 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     52817  0.0  0.0  61364  1148 ?        Ss    2014   0:00 /usr/sbin/sshd -D
    root     61143  0.0  0.0  61364  1144 ?        Ss   Feb10   0:00 /usr/sbin/sshd -D
    root     63108  0.0  0.0  61364  1184 ?        Ss   Jan13   0:00 /usr/sbin/sshd -D
    root     64920  0.0  0.0  61364  1148 ?        Ss    2014   0:00 /usr/sbin/sshd -D

因此发觉docker中的所有进程，在宿主机中是可见的，这样比较容易误操作。

Docker容器调优
-----------------------------------
我先抛出问题，我们Base组利用docker进行部署几个服务，包括Base服务，NeunnManager服务，NeunnPortal服务，但是问题是经常发现docker中的tomcat无缘无故的自动退出，当然，这里也有OutOfMemory和OutOfPermgenSpace，但是这两个问题可以通过Tomcat参数调优进行解决，也可以进行Docker的参数调优。

但是自动退出这个问题，由于没有合适的监控，没有任何日志信息，这里没有任何解决办法，目前的策略是将每个服务进行彻底分离，并将Bamboo的Agent与服务部署的容器进行分离，避免相互干扰。


Docker参考手册
===================================
这里填写一些命令参考，供翻阅。

Using Docker
-----------------------------------
Install docker on OpenSuse:

::

 $sudo zypper ar -f http://download.opensuse.org/repositories/Virtualization/openSUSE_13.1/ Virtualization
 $sudo rpm --import http://download.opensuse.org/repositories/Virtualization/openSUSE_13.1/repodata/repomd.xml.key
 $ssudo zypper in docker
 $sudo systemctl start docker
 $sudo systemctl enable docker(optional)

Example:

::
 
    $sudo docker run [option] [imagename] [command]
    $sudo docker run -t -i ubuntu:14.04 /bin/bash (-t means create a terminal, -i means we can interact with stdin)
    $sudo docker run -d ubuntu:14.04 /bin/bash (-d means run in deamon process)
    $sudo docker run -t -i -p localhost:8080:80 ubuntu:14.04 /bin/bash(port mapping the container 80 port to host 8080 port)
    $docker attach [containerId]
    $docker logs [containerId]
    $docker commit [containerId] name/imagename:versionId

NAT with iptables:

::

    iptables -t nat -A  DOCKER -p tcp --dport   <local port> -j DNAT --to-destination <docker ip>:<docker port>

Docker 开机自启动tomcat服务
-----------------------------------
这里的镜像是从tumtu下载的带有ssh服务的ubuntu镜像，他的dockerfile如下：

::

    FROM ubuntu:latest
    MAINTAINER Knight/basic:0.1<yangwj@neunn.com> 

    # Install packages
    RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install openssh-server pwgen
    RUN mkdir -p /var/run/sshd && sed -i "s/UsePrivilegeSeparation.*/UsePrivilegeSeparation no/g" /etc/ssh/sshd_config && sed -i "s/UsePAM.*/UsePAM no/g" /etc/ssh/sshd_config && sed -i "s/PermitRootLogin.*/PermitRootLogin yes/g" /etc/ssh/sshd_config
    ADD jdk1.7.0_55 /usr/lib/jdk1.7.0_55
    ADD tomcat7 /usr/local/tomcat7
    ADD profile /etc/profile
    ADD hosts /etc/hosts
    ADD set_root_pw.sh /set_root_pw.sh
    ADD run.sh /run.sh
    ENV JAVA_HOME /usr/lib/jdk1.7.0_55
    RUN chmod +x /*.sh

    EXPOSE 22 8888
    CMD ["/run.sh"]




reference
-----------------------------------
http://www.widuu.com/chinese_docker/installation/opensuse.html
http://www.pchou.info/open-source/2014/03/29/docker-introduction.html

