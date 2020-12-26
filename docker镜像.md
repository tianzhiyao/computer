#### 1、查看镜像信息
```
#查看完整镜像
[root@localhost ~]# docker inspect centos:latest 
#查看完整的镜像ID
[root@localhost ~]# docker images --no-trunc  
#进入容器
[root@localhost ~]# docker run -it centos:latest /bin/bash 
```
#### 2、创建自己的image


```
#进入容器
[root@localhost ~]# docker run -it centos:latest /bin/bash
#安装mariadb
[root@6cd9e03fff90 /]# yum -y install mariadb-server
#退出并记住ID号
[root@6cd9e03fff90 /]# exit
exit
#提交相应的副本
[root@localhost ~]# docker commit -a tianyao -m "add mariadb" 6cd9e03fff90 mynariadb:v2
sha256:1cf5b43cff1ad0090c46e42b94aba4deac0918009eecfc11b02c0eef6906c160
 -a ：可以指定更新的用户信息
 -m ：指定提交的说明信息； 之后是用来创建镜像的容器ID；最后指定目标镜像的仓库名和tag信息，创建成功后会发挥这个镜像的ID信息
#查看新创建的镜像是否存在
[root@localhost ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
mynariadb    v2        1cf5b43cff1a   20 seconds ago   469MB
#进入新创建的容器
[root@localhost ~]# docker run -it mynariadb:v2 /bin/bash
#查看是否有包
[root@11072c2a6ce7 /]# rpm -qa | grep mariadb
#退出后进入同一个容器并不会进入同一个环境
```
#### 3、dockerfile创建image
```
#1、从本地文件系统导入
#下载镜像
https://wiki.openvz.org/Download/template/precreated
#导入镜像
[root@localhost ~]# docker image import centos-6-x86_64.tar.gz centos6
[root@localhost ~]# docker images
[root@localhost ~]# docker run -it centos6 /bin/bash
#查看版本
[root@localhost ~]# cat /etc/redhat-release
#查看内核
[root@localhost ~]# uname -a
```
#### 4、上传镜像
```
#登录docker
[root@localhost ~]# docker login
#给要上传的镜像打一个标签并起一个名字
[root@localhost ~]# docker tag nginx:latest tianyao/mynginx:latest
#上传镜像
[root@localhost ~]# docker push tianyao/mynginx:latest
```
#### 5、移除本地image
```
#移除镜像
[root@localhost ~]# docker rmi  tianyao/mynginx
加 -f 强制移除
```
#### 6、存入或导出镜像
```
#打包
[root@localhost ~]# docker save -o myimages.tar.gz centos nginx mynariadb
#删除所存在的镜像
[root@localhost ~]# docker rmi -f centos mynariadb:v2 nginx
#导入镜像
[root@localhost ~]# docker load --input myimages.tar.gz
#查看
[root@localhost ~]# docker images
```
