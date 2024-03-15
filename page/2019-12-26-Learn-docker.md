---
title: Docker 学习笔记
date:  	2019-12-26 18:56:36 +0800
category: Original
tags: [Docker,Learning]
excerpt: 用于长期可更新的Docker学习笔记
---

### 引入

图书馆项目部署：

1. 利用`MobaXterm`工具连接服务器，将war包放入root文件夹
2. 输入命令 `docker cp 文件名.war 容器:/usr/local/tomcat/webapps`
3. 重启容器 `docker restart 容器`

tomcat容器崩坏无法部署需要修改`server.xml`：

1. 从容器里面拷文件到宿主机 `docker cp 容器:/usr/local/tomcat/conf/server.xml /root`
2. 在本地修复`server.xml`后再放入宿主机root文件夹
3. 从宿主机拷文件到容器 `docker cp server.xml 容器:/usr/local/tomcat/conf`
4. 启动容器 `docker start 容器`

### 常用功能

#### 拉取tomcat镜像

```
docker pull tomcat/io
```

#### 生成tomcat容器

```
docker run -it --name myTomcat -d -p 8080:8080 tomcat
```

#### 拉取mysql镜像

```
docker pull mysql:latest
```

#### 生成mysql容器

```
docker run -it --name myMysql -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=myPassword mysql
```

参数说明：

**-p 3306:3306** ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 **宿主机ip:3306** 访问到 MySQL 的服务。

**MYSQL_ROOT_PASSWORD=123456**：设置 MySQL 服务 root 用户的密码

**错误**

使用`Navicat Premium` 连接MySQL时出现如下错误：2059

**原因**

mysql8 之前的版本中加密规则是`mysql_native_password`,而在mysql8之后,加密规则是`caching_sha2_password`

**解决**

```
更改加密规则：mysql -uroot -password #登录
use mysql; #选择数据库# 远程连接请将'localhost'换成'%'
ALTER USER 'root'@'%' IDENTIFIED BY 'myPassword' PASSWORD EXPIRE NEVER; #更改加密方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'myPassword'; #更新用户密码
FLUSH PRIVILEGES; #刷新权限
```

#### 更多镜像

docker官方镜像仓库`https://hub.docker.com/`

#### 删除镜像

```
docker rmi 镜像id
```

#### 从容器里面拷文件到宿主机

```
docker cp 容器:容器的路径 宿主机的路径
```

#### 从宿主机里面拷文件到容器

```
docker cp 宿主机的路径 容器:容器的路径
```

#### 查看镜像

```
docker images
```

#### 查看容器

```
docker ps
```

#### 进入正在运行的容器

```
docker exec -it 容器 /bin/bash
```

#### 启动容器

```
docker start 容器
```

#### 查看容器日志文件

```
docker logs 容器
```

#### 终止容器

```
docker stop 容器
```

#### 重启容器

```
docker restart 容器
```

#### 删除容器

```
docker rm 容器
```

#### 导出容器

```
docker export 容器 > default.tar
```

#### 导入容器

```
docker import default.tar 镜像
```

#### 拉取redis

```
docker pull redis:latest
```

#### 生成redis容器

```
docker run -it --name redis-blog -d -p 6379:6379 redis
```

#### 拉取**elasticsearch**

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.7.2
```

#### 生成**elasticsearch**容器

```
docker run -d -p 9200:9200 -p 9300:9300 --name="elastic-blog" -e ES_JAVA_OPTS="-Xms256m -Xmx256m" docker.elastic.co/elasticsearch/elasticsearch:6.7.2
```

#### 拉取rabbitMQ

```
docker pull rabbitmq:management
```

#### 生成rabbitMQ容器

```
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq-blog rabbitmq:management
```

#### 