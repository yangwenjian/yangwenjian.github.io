


===========================================
Shell
===========================================
Shell is the most powerfull tool in linux, no one of them.(最强工具，没有之一^-^)

Shell
===========================================
密码的艺术

我们在使用服务器时经常喜欢用带有!@#$的某些字符作为密码，这带来一个问题，shell中!是有意义的，表示上一条命令。
这里有两种方式解决这个问题：

进行转意，例如：

::

    mysqldump -uroot -p"Hello@"\!"$" -A > dump.sql

或者使用脚本进行变量转换，例如：

::

    dumpfile=$(date +%Y%m%d%H%M).sql
    dbuser=root
    dbpassword='Hello@!$'
    mysqldump -u${dbuser} -p${dbpassword} -A > ${dumpfile}


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

This is a shell check gmail.

::
    
    #!/bin/bash

    curl -u yangwenjian99@gmail.com --silent "https://mail.google.com/mail/feed/atom" | perl -ne \
        '
    print "Subject: $1 " if /<title>(.+?)<\/title>/ && $title++;
    print "(from $1)\n" if /<email>(.+?)<\/email>/
        '

This is an example to check network.

::

    #!/bin/bash

    for((i=1;i<=255;i++));do
        ping -c 5 192.168.1.$i
        done

Here is a pit in my work. I want to count the first line of one file, but it doesn't work.

::

    count=0
    sum=0
    awk '{print $1}' public/shell/testfile | while read onehelloline
    do
        echo $onehelloline
        ((count= count+1))
        echo $count
        ((sum= sum+onehelloline))
        echo $sum
    done
    echo $count
    echo $sum
    averagetime=`expr $(($sum/$count))`
    echo $averagetime
     
最后发现count和sum都是0,原因是那块循环中并没有影响到全局变量。修改循环变量如下：

::

    data1=`awk '{print $1}' public/shell/testfile`
    for data in $data1;do
        ((count= count+1))
        ((sum= sum+data))
    done

