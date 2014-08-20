


===========================================
Shell
===========================================
Shell is the most powerfull tool in linux, no one of them.(最强工具，没有之一^-^)

Awk
===========================================
Awk is very popular in linux operation. Awk process with text in architecutre. 
It defines every line one record, and defines every string one field, which is devided by something with others.

Some examples:

打印第一行：    awk ‘NR==1{print $0}'

打印第一行最后一个字段：    awk -F: 'NR==1{print $NF}'


This is a shell use awk to start all the docker container when the power is on. I put this shell to excute in /etc/rc.local.

::

 #!/bin/bash
  
 list=`docker ps -a | awk 'NR!=1{print $1}'` 
 echo ${list}

 for container in $list 
     do
     docker start $container
     done
