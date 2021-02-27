## 创建三节点的swarm集群

角色 | IP
---|---
swarm-manage | 192.168.43.10
swarm-worker1 | 192.168.43.11
swarm-worker1 | 192.168.43.12


### 1、编辑hosts文件
```
[root@manager ~]# vim /etc/hosts
192.168.43.10 swarm-manage
192.168.43.11 swarm-worker1
192.168.43.12 swarm-worker2
```
### 2、关闭selinux防火墙，并开启防火墙端口
```
[root@worker1 ~]# sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
[root@worker1 ~]# setenforce 0
[root@worker2 ~]# firewall-cmd --add-port={2377,4789,7946}/tcp --permanent
[root@worker2 ~]# firewall-cmd --add-port={4789,7946}/udp --permanent
[root@worker2 ~]# firewall-cmd --reload
```
### 3、三台都设置免密
```
[root@manager ~]# ssh-keygen -t rsa
[root@manager ~]# ssh-copy-id 192.168.43.11
[root@manager ~]# ssh-copy-id 192.168.43.12
```
### 4、准备docker镜像
```
[root@worker2 ~]# docker pull nginx
[root@worker2 ~]# docker pull centos
[root@manager ~]# docker pull busybox
[root@manager ~]# docker pull reqistry # 私有仓库
```
### 5、配置私有仓库
```
[root@manager ~]# docker run -dit --name registry -p 5000:5000 -v /var/registry:/var/lib/registry --restart always registry
[root@manager ~]# vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://qldpdx6i.mirror.aliyuncs.com"],
  "insecure-registries": ["192.168.43.10:5000"]
}

将配置文件传到另外两个节点
[root@manager ~]# scp /etc/docker/daemon.json 192.168.43.11:/etc/docker/
[root@manager ~]# scp /etc/docker/daemon.json 192.168.43.12:/etc/docker/
所有节点重新启动docker
[root@manager ~]# systemctl daemon-reload
[root@manager ~]# systemctl restart docker


做标签
[root@manager ~]# docker tag busybox:latest 192.168.43.10:5000/busybox
[root@manager ~]# docker tag nginx:latest 192.168.43.10:5000/nginx
[root@manager ~]# docker tag centos:latest 192.168.43.10:5000/centos

上传镜像
[root@manager ~]# docker push 192.168.43.10:5000/busybox
[root@manager ~]# docker push 192.168.43.10:5000/nginx
[root@manager ~]# docker push 192.168.43.10:5000/centos

查看私有仓库
[root@manager ~]# curl 192.168.43.10:5000/v2/_catalog
{"repositories":["busybox","centos","nginx"]}


```
### 6、创建添加swarm集群
```
初始化集群
[root@manager ~]# docker swarm init --advertise-addr 192.168.43.10

查看token
[root@manager ~]# docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0oqtgr47imi3wou6kwq1vqb6ersxh690trzh29dgfg6alb2iz0-1gls2vp96zz2loqsn8nwn62zs 192.168.43.10:2377

在其他节点上添加进集群
docker swarm join --token SWMTKN-1-0oqtgr47imi3wou6kwq1vqb6ersxh690trzh29dgfg6alb2iz0-1gls2vp96zz2loqsn8nwn62zs 192.168.43.10:2377

查看节点
[root@manager ~]# docker node ls


将其他节点提升为manage
[root@manager ~]# docker node promote worker1

将节点降级
[root@manager ~]# docker node domote worker1
```
### 7、图形化
```
部署可视化界面
[root@manager ~]# docker pull dockersamples/visualizer
[root@manager ~]# docker tag dockersamples/visualizer:latest 192.168.43.10:5000/visualizer
[root@manager ~]# docker push 192.168.43.10:5000/visualizer
[root@manager ~]# docker run -d -p 8080:8080 -e HOST=172.16.0.10 -e PORT=8080 -v /var/run/docker.sock:/var/run/docker.sock --name visualizer 192.168.43.10:5000/visualizer
在浏览器上访问192.168.43.10:5000端口
```
### 8、操作命令
```
开启服务
[root@manager ~]# docker service scale web1=3

删除容器
[root@manager ~]# docker service rm web1

直接开启5个
[root@manager ~]# docker service create --name web2 --replicas 5 192.168.43.10:5000/nginx

不希望在 manager 上运行service
[root@manager ~]# docker node update --availability drain manager

发布服务
[root@manager ~]# docker service update --publish-add 9999:80 web2

直接带带端口的开启
[root@manager ~]# docker service create --name web2 --replicas 5 --publish 7777:80 192.168.43.10:5000/nginx

在浏览器上访问7777端口
```