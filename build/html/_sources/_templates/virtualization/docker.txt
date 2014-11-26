


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

Docker实践
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

build后产生新的镜像，结果怎么run这个镜像也跑不起来，直接镜像就推出，通过docker logs也看不出什么。

运行我镜像列表的里的所有镜像，发现都是同一个毛病，求助于同事，同事查看了一通后也没发现明显的问题。他只是觉得镜像有问题，最后发现是我在构建的时候下载镜像的过程中断网了，结果镜像没有下去，有问题，当时就被公司的网络耍了一把。

用了新的镜像后发现docker file有些内容没有写进去，profile是写进去了，但是tomcat和jdk都没有进入文件系统中，其实是我的dockerfile写法有问题，ADD添加文件夹的时候和我们观念上的添加文件夹不同，需要给文件夹指定名称。正确的写法如下：

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

Run docker:

::
 
 $sudo docker run [option] [imagename] [command]
 $sudo docker run -t -i ubuntu:14.04 /bin/bash (-t means create a terminal, -i means we can interact with stdin)
 $sudo docker run -d ubuntu:14.04 /bin/bash (-d means run in deamon process)
 $sudo docker run -t -i -p localhost:8080:80 ubuntu:14.04 /bin/bash(port mapping the container 80 port to host 8080 port)

Usual command:

::

 Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from a container's filesystem to the host path
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    export    Stream the contents of a container as a tar archive
    history   Show the history of an image
    images    List images
    import    Create a new filesystem image from the contents of a tarball
    info      Display system-wide information
    inspect   Return low-level information on a container
    kill      Kill a running container
    load      Load an image from a tar archive
    login     Register or log in to the Docker registry server
    logs      Fetch the logs of a container
    port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
    pause     Pause all processes within a container
    ps        List containers
    pull      Pull an image or a repository from a Docker registry server
    push      Push an image or a repository to a Docker registry server
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image to a tar archive
    search    Search for an image on the Docker Hub
    start     Start a stopped container
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Lookup the running processes of a container
    unpause   Unpause a paused container
    version   Show the Docker version information
    wait      Block until a container stops, then print its exit code
    
Example:

::

    $docker ps -a
    $docker start [containerId]
    $docker attach [containerId]
    $docker stop [containerId]
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

