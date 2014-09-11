


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

reference
-----------------------------------
http://www.widuu.com/chinese_docker/installation/opensuse.html
http://www.pchou.info/open-source/2014/03/29/docker-introduction.html

