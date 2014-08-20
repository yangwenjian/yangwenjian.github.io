


=====================================
Docker
=====================================
Docker is a virtualization tool Based on Linux Container(LXC).
Useful and easy-to-use.

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


reference
-----------------------------------
http://www.widuu.com/chinese_docker/installation/opensuse.html
http://www.pchou.info/open-source/2014/03/29/docker-introduction.html

