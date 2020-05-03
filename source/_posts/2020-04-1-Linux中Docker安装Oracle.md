---
title: Docker 中安装 Oracle
date: 2019-09-25 09:25:00
top: true  推荐文章（文章是否置顶），如果 top 值为 true，则会作为首页推荐文章
categories: Docker
tags:
  - Docker
  - java
---

**摘要：** 文章主要讲解在linux中如何在docker上部署安装Oracle数据库软件
<!-- more -->

### 1. 安装前准备

在进行Oracle的安装时我们首先需要安装Docker容器，关于Oracle的安装我们在之前的文章中有，如果不懂Docker的可以先看这篇文章：[Docker 的安装与简单命令](https://i-code.online/2019/09/14/Docker%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/) 

好了我们前期环境都准备好了，可进入正题了~

### 2. Oracle的安装  

* 通过 docker search oracle 可以查看查看有哪些资源

``` 
[root@localhost ~]# docker search oracle
NAME                                  DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
oraclelinux                           Official Docker builds of Oracle Linux.         642                 [OK]                
jaspeen/oracle-11g                    Docker image for Oracle 11g database            156                                     [OK]
oracleinanutshell/oracle-xe-11g                                                       89                                      
oracle/graalvm-ce                     GraalVM Community Edition Official Image        60                                      [OK]
oracle/openjdk                        Docker images containing OpenJDK Oracle Linux   60                                      [OK]
absolutapps/oracle-12c-ee             Oracle 12c EE image with web management cons…   38                                      
araczkowski/oracle-apex-ords          Oracle Express Edition 11g Release 2 on Ubun…   29                                      [OK]
oracle/nosql                          Oracle NoSQL on a Docker Image with Oracle L…   23                                      [OK]
bofm/oracle12c                        Docker image for Oracle Database                23                                      [OK]
datagrip/oracle                       Oracle 11.2 & 12.1.0.2-se2 & 11.2.0.2-xe        18                                      [OK]
...
...
```

* 我们这里直接用 `docker pull 镜像名` 获取镜像

我们指定了用阿里的 `registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g` 

``` 
[root@localhost ~]# docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
Using default tag: latest
latest: Pulling from helowin/oracle_11g
ed5542b8e0e1: Pull complete 
a3ed95caeb02: Pull complete 
1e8f80d0799e: Pull complete 
Digest: sha256:4c12b98372dfcbaafcd9564a37c8d91456090a5c6fb07a4ec18270c9d9ef9726
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g:latest
registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g:latest

```

* 等待拉取镜像，等成功后我们可以通过 `doker images` 查看本地镜像

``` 
[root@localhost ~]# docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g   latest              3fa112fd3642        4 years ago         6.85GB

```  

* 创建容器应用 通过 `docker run` 命令，关于命令的具体使用请参考之前文章[Docker 的安装与简单命令](https://i-code.online/2019/09/14/Docker%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/) 

```

[root@localhost ~]# docker run -d -p 1521:1521 --name oracle_11g registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
2594effc414c8301bf1fbf41fe21388b2afbaa72993e7b7cea3850663c37972d

``` 

创建完后我们通过 `docer ps` 查看运行的容器情况  
```

[root@localhost ~]# docker ps
CONTAINER ID        IMAGE                                                  COMMAND                  CREATED              STATUS              PORTS                    NAMES
2594effc414c        registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g   "/bin/sh -c '/home/o…"   About a minute ago   Up About a minute   0.0.0.0:1521->1521/tcp   oracle_11g

``` 

> 到这里我们的Oracle创建基本结束了，接下来我们进行Oracle的设置和启动

### 3. Oracle容器的启动与设置

* 启动容器 `docker start 容器名字` 

```

[root@localhost ~]# docker start oracle_11g
oracle_11g

``` 

* 进入镜像

> docker exec -it 容器名 bash

```

[root@localhost ~]# docker exec -it oracle_11g bash
[oracle@2594effc414c /]$ 

``` 

* 进行相应的配置

 

 1. 切换到 `root` 用户 `su root` ,这里密码是 `helowin` 

```

[oracle@2594effc414c /]$ su root 
Password: 
[root@2594effc414c /]#

``` 

2. 编辑 `profile` 文件配置 `ORACLE` 环境变量

```

[root@2594effc414c /]# vi /etc/profile

``` 
在文件末尾加入下面内容：
```

export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=ORACLEHOME/bin: ORACLE_HOME/bin: ORACLE

``` 

3. 创建软连接，并返回Oracle用户

```

[root@2594effc414c /]# ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
[root@2594effc414c /]# su - oracle
[oracle@2594effc414c ~]$ 

``` 

4. 登录sqlplus–修改sys、system用户密码–创建用户 

```

[oracle@2594effc414c ~]$ sqlplus /nolog

SQL*Plus: Release 11.2.0.1.0 Production on Thu Apr 9 10:47:56 2020

Copyright (c) 1982, 2009, Oracle. All rights reserved.
SQL> conn /as sysdba
Connected.
SQL> 

``` 

主要命令：
```

修改密码：
alter user system identified by system; 
alter user sys identified by sys; 
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED; 
创建用户
create user test identified by test; 
并给用户赋予权限
grant connect, resource, dba to test; 

``` 
如下：
```

SQL> conn /as sysdba
Connected.
SQL> alter user system identified by system; 

User altered.

SQL> alter user sys identified by sys; 

User altered.

SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED; 

Profile altered.

SQL> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

```
到这里位置我们就配置结束了，接下来可用工具来连接了，这里我们用 `DataGrip` 来连接测试 

![img](https://raw.githubusercontent.com/AnonyStar/AnonyStar.github.io/master/images/post\docker/datagrip-oracle.png)

