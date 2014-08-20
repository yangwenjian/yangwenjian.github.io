


=========================================
Project Management Tools
=========================================

Project Management
=========================================
Project Management is an important part in software engineering.We use it to control the process of our software development.


Atlassian Confluence Wiki
-----------------------------------------
Confluence is a good document online.

Atlassian Bamboo
-----------------------------------------
bamboo作为自动化构建项目的工具，需要进行一些项目任务配置。

1.创建项目，点击create->new project；

2.配置git，使bamboo获取项目工程，选择右上角linked repositories，点击add repository，配置git地址，并选择分支和填写用户名密码；

3.创建plan，点击create->new plan；

4.利用Docker创建容器并部署Agent；

    4.1.创建容器并部署agent，docker run -i -t -p 192.168.250.3:8887:8888 BambooAgent6/ubuntu:14.04 /startBambooAgent.sh；
  
    4.2.创建task，点击All build plan，点击右边铅笔，进入task配置页面，点击default job进行自动化任务配置（这里以base层工程为例）
    
        4.2.1点击add task，选择source code checkout，选择相应reposity，点击Save；
        
        4.2.2点击add task，选择maven3.X，在Goal中输入“clean install”（清理并构建）并指定working sub directory为ncloud-base-rp-conn（否则找不到构建工程）；
        
        4.2.3点击add task，选择maven3.X，在Goal中输入“clean install”（清理并构建）并指定working sub directory为ncloud-base；
        
        4.2.4点击add task，选择stop tomcat， 在tomcat的URL中输入http://172.17.0.4:8888/manager/并输入tomcat的管理员的帐号和密码（这里需要在tomcat进行一下配置），并制定application context为“/ncloud”；
         
        4.2.5点击add task，选择deploy tomcat application，配置和4.2.4相同，除了war file中指定base层的war包“/ncloud-base/target/ncloud-base-0.0.1-SNAPSHOT.war”；
        
        4.2.6点击add task，选择start tomcat，配置如4.2.4；
    
5.创建容器并运行Agent，在宿主机上利用docker进行容器创建 docker run -i -t -p 192.168.250.3:8887:8888 BambooAgent6/ubuntu:14.04 /startBambooAgent.sh；

6.配置Agent，右上角进入agents的配置页面，选项卡选中agent authorization，选择创建好的agent进行approve access，授权后，可以在online remote agents中看到相应的agent。这里需要注意，我们建议先进行任务创建再创建相应agent，如果先建立agent，可能无法进行自动构建，只能重新创建任务。

7.运行计划任务，点击build all plan中的开始按钮，即可享受全自动的构建测试服务，之后看相应的报告即可。
                                                                                                                                                                             
*鸣谢：*

崔远智，bamboo使用的提倡者，解决bamboo任务运行中的坑，并发扬光大项目管理工具。

黄金龙，首先吃螃蟹的人，解决工程配置中的坑，进行初始化的各种配置。

**对于Linux和Mac用户，可以参考下面的脚本进行本地构建并部署到服务器上，Windows用户自求多福吧。**
::

 #!/bin/bash
 #this is a deploy shell for base to 192.168.250.3
 cd /home/knight/code/workspace/base/ncloud-base-rp-conn
 mvn clean
 mvn install

 cd /home/knight/code/workspace/base/ncloud-base
 mvn clean
 mvn install

 scp /home/knight/.m2/repository/Neunn/rest/base/ncloud-base/0.0.1-SNAPSHOT/ncloud-base-0.0.1-SNAPSHOT.war root@192.168.250.3:~/ncloud.war

 ssh root@192.168.250.3 << EOF
 cd /usr/local/tomcat7/
 rm -rf webapps/ncloud*
 cp /root/ncloud.war webapps/
 ./bin/shutdown.sh
 ./bin/startup.sh
 EOF
                                                                                                                                                                        
参考:
https://confluence.atlassian.com/display/BAMBOO/Bamboo+Documentation+Home

