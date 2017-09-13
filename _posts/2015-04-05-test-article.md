---
layout: postcn
title: "docker mysql主从复制"
date: 2017-09-10 08:00:00 +0800
lang: cn
nav: post
category: test
tags: [test, article]
---

* content
{:toc}

1、	docker已经在centos6.5（或者其他操作系统）上安装好
docker -v
Docker version 1.7.1, build 786b29d/1.7.1



<!-- more -->
1、	docker已经在centos6.5（或者其他操作系统）上安装好
docker -v
Docker version 1.7.1, build 786b29d/1.7.1
2、	拉取镜像
docker pull hub.c.163.com/library/mysql:5.7
docker images  #查看是否有镜像列表
3、	启动一个容器
在宿主机路径下新建一个目录用来和docker容器共享数据：
mkdir –p /home/mysql/master-data
启动容器：
docker run \
--name=mysql-master \
--hostname=mysql \
-v /home/mysql/master-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD='123456' \
-d hub.c.163.com/library/mysql:5.7

--name=指定容器名字
--hostname=主机名
-v 宿主机路径：容器内路径 
-e 环境变量
-d detach模式

宿主机路径必须事先存在
4、	配置master
如何进入刚刚启动的那个容器？
docker exec -it mysql-master bash 
这样就进入到一个装好mysql的容器中了，你可以执行命令试试：
mysql –u root –p
输入密码
show databases;
use mysql;
show tables;
exit;










现在我们要修改下mysql的配置，作为master
cd /etc/mysql/conf.d
在这个目录下修改或者新增my.cnf(用vi命令)
由于mysql官方镜像没有安装vi，我们还需先安装：
apt-get update && apt-get install vim
安装好后，执行：vi my.cnf
输入内容如下：


[mysqld]
server-id=1
log-bin=mysql-bin


选作：可以commit这个容器状态为镜像，以后可以复用
docker commit mysql-master mysql/master:1.0
5、	重启master
到宿主机上执行：
docker restart mysql-master
或者启动新镜像：
docker run \
--name=mysql-master \
-v /home/mysql/master-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD='123456' \
-d mysql/master:1.0

查看主mysql信息：
mysql –u root –p
输入密码

mysql> show master status
    -> ;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

记下file和position的值

6、	宿主机，启动另一个容器
docker run --name mysql-slave1 \
–link mysql-master:master \
-v /home/mysql/slave1-data/:/var/lib/mysql/ -d mysql/master:1.0

注：–link mysql-master:master \  链接到主mysql。主mysql的容器地址可以用master表示
7、	配置slave，同第4步，进入到mysql-slave1
不同的是设定server-id=2
然后重启：
docker restart mysql-slave1

这时MySQL的主从服务器都在运行中，需要读者自行保证两边的数据相同。只有当两台服务器的数据都一样的时候，才能建立主从复制连接。

8、
配置从库连接主库, 在从库上执行 
mysql> change master to master_host='master', master_port=3306, master_user='root', master_log_file='mysql-bin.000001', master_log_pos=154;        
最后两项
MASTER_LOG_FILE 和  MASTER_LOG_POS 就是刚才我们在master上记录下来的值

在主库执行 : SHOW MASTER STATUS; 命令可以取得
 
对应的字段是 File 和 Position
在从库启动 slave 线程开始同步
START SLAVE;
在从库 查看同步状态
show slave status;
如果看到 Slave_Io_State 字段有 :
Waiting for master to send event ... 
那就成功了 ! ! !

8、	验证
只要在主服务器数据库插入或修改数据，就可以自动复制到从服务器上了。
