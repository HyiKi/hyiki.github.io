---
title: MongoDB 分片集群
date:  	2020-03-14 08:56:36 +0800
category: Original
tags: MongoDB
excerpt: 横向扩展提高数据库性能的一种环境搭建
---


## 前言

前期认识：切分属于一种横向扩展

[切分](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)是一种在多台计算机之间分发数据的方法。MongoDB使用切分来支持具有非常大的数据集和高吞吐量操作的部署。

具有大数据集或高吞吐量应用程序的数据库系统可能会挑战单个服务器的容量。例如，较高的查询速率会耗尽服务器的CPU容量。大于系统RAM的工作集大小强调磁盘驱动器的I/O能力。

解决系统增长的方法有两种：**垂直**和**横向**扩展。

*垂直标度*涉及到**增加单个服务器的容量**，**例如使用更强大的CPU**、**添加更多的RAM或增加存储空间**。可用技术的限制可能会限制单个机器对给定的工作负载具有足够的功能。此外，基于云的供应商有基于可用硬件配置的硬天花板.因此，有一个实际的最大垂直缩放。

*水平标度*涉及**将系统数据集划分为多个服务器并加载**，**添加额外的服务器以根据需要增加容量**。虽然单个机器的总速度或容量可能不高，但每台机器都处理整个工作负载的一个子集，可能比单个高速高速高容量服务器提供更高的效率。扩展部署容量只需要根据需要添加额外的服务器，这可能比单个计算机的高端硬件成本更低。交换条件是增加了部署的基础设施和维护的复杂性。

## 本次部署的配置服务和分片

```
/mongos0 - 172.17.0.7
/shardsvr12 - 172.17.0.14
/shardsvr11 - 172.17.0.13
/shardsvr10 - 172.17.0.12
/shardsvr02 - 172.17.0.11
/shardsvr01 - 172.17.0.10
/shardsvr00 - 172.17.0.9
/configsvr2 - 172.17.0.8
/configsvr1 - 172.17.0.6
/configsvr0 - 172.17.0.3
```

**查看启动的容器和对应的ip列表**

对应的指令`docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)`

## 一、创建配置服务复制集

```
//创建配置服务复制集
docker run --name configsvr0 -d mongo --configsvr --replSet "rs_configsvr"  --bind_ip_all
docker run --name configsvr1 -d mongo --configsvr --replSet "rs_configsvr"  --bind_ip_all
docker run --name configsvr2 -d mongo --configsvr --replSet "rs_configsvr"  --bind_ip_all
```

### 通过docker inspect 容器名去获取在容器里的ip地址：

```
docker inspect configsvr0 | grep IPAddress
docker inspect configsvr1 | grep IPAddress
docker inspect configsvr2 | grep IPAddress
```

显示结果：

```

[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect configsvr0 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect configsvr1 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.6",
                    "IPAddress": "172.17.0.6",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect configsvr2 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.8",
                    "IPAddress": "172.17.0.8",

```

由于–configsvr 的默认端口为 27019。所以配置服务的地址为

- configsvr0: 172.17.0.3:27019
- configsvr1: 172.17.0.6:27019
- configsvr2: 172.17.0.8:27019

### 初始化配置服务复制集：

```
docker exec -it configsvr0 bash
mongo --host 172.17.0.3 --port 27019
```

此时，进入了configsvr0的mongo，输入命令：

```
rs.initiate(
	{
		_id:"rs_configsvr",
		configsvr: true,
		members: [
			{ _id : 0, host : "172.17.0.3:27019" },
			{ _id : 1, host : "172.17.0.6:27019" },
			{ _id : 2, host : "172.17.0.8:27019" }
		]
	}
)
```

输出结果，配置服务复制集成功：

```
{
        "ok" : 1,
        "$gleStats" : {
                "lastOpTime" : Timestamp(1584167961, 1),
                "electionId" : ObjectId("000000000000000000000000")
        },
        "lastCommittedOpTime" : Timestamp(0, 0),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584167961, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1584167961, 1)
}
```

连接字符串为

```
mongodb://172.17.0.3:27019,172.17.0.6:27019,172.17.0.8:27019/test?replicaSet=rs_configsvr
```

## 二、创建分片复制集

```
//创建分片0复制集
docker run --name shardsvr00 -d mongo --shardsvr --replSet "rs_shardsvr0"  --bind_ip_all
docker run --name shardsvr01 -d mongo --shardsvr --replSet "rs_shardsvr0"  --bind_ip_all
docker run --name shardsvr02 -d mongo --shardsvr --replSet "rs_shardsvr0"  --bind_ip_all
//创建分片1复制集
docker run --name shardsvr10 -d mongo --shardsvr --replSet "rs_shardsvr1"  --bind_ip_all
docker run --name shardsvr11 -d mongo --shardsvr --replSet "rs_shardsvr1"  --bind_ip_all
docker run --name shardsvr12 -d mongo --shardsvr --replSet "rs_shardsvr1"  --bind_ip_all
```

### 通过docker inspect 容器名去获取在容器里的ip地址：

```
docker inspect shardsvr00 | grep IPAddress
docker inspect shardsvr01 | grep IPAddress
docker inspect shardsvr02 | grep IPAddress
docker inspect shardsvr10 | grep IPAddress
docker inspect shardsvr11 | grep IPAddress
docker inspect shardsvr12 | grep IPAddress
```

显示结果：

```
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect shardsvr00 | grep IPAddress
docker inspect shardsvr01 | grep IPAddress
docker inspect shardsvr02 | grep IPAddress
docker inspect shardsvr10 | grep IPAddress
docker inspect shardsvr11 | grep IPAddress
docker inspect shardsvr12 | grep IPAddress            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.9",
                    "IPAddress": "172.17.0.9",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect shardsvr01 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.10",
                    "IPAddress": "172.17.0.10",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect shardsvr02 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.11",
                    "IPAddress": "172.17.0.11",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect shardsvr10 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.12",
                    "IPAddress": "172.17.0.12",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect shardsvr11 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.13",
                    "IPAddress": "172.17.0.13",
[root@iZwz95jecxvhuu0q9m3yu4Z ~]# docker inspect shardsvr12 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.14",
                    "IPAddress": "172.17.0.14",
```

由于–shardsvr 的默认端口为 27018。所以配置服务的地址为

- shardsvr00: 172.17.0.9:27018
- shardsvr01: 172.17.0.10:27018
- shardsvr02: 172.17.0.11:27018
- shardsvr10: 172.17.0.12:27018
- shardsvr11: 172.17.0.13:27018
- shardsvr12: 172.17.0.14:27018

### 初始化分片复制集：

```
docker exec -it shardsvr00 bash
mongo --host 172.17.0.9 --port 27018
```

此时，进入了shardsvr00的mongo，输入命令：

```
rs.initiate(
	{
		_id:"rs_shardsvr0",
		members: [
			{ _id : 0, host : "172.17.0.9:27018" },
			{ _id : 1, host : "172.17.0.10:27018" },
			{ _id : 2, host : "172.17.0.11:27018" }
		]
	}
)
```

输出结果，配置分片0复制集成功：

```
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584168060, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1584168060, 1)
}
```

------

```
docker exec -it shardsvr10 bash
mongo --host 172.17.0.12 --port 27018
```

此时，进入了shardsvr10的mongo，输入命令：

```
rs.initiate(
	{
		_id:"rs_shardsvr1",
		members: [
			{ _id : 0, host : "172.17.0.12:27018" },
			{ _id : 1, host : "172.17.0.13:27018" },
			{ _id : 2, host : "172.17.0.14:27018" }
		]
	}
)
```

输出结果，配置分片1复制集成功：

```
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584168192, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1584168192, 1)
}
```

连接字符串为

```
mongodb://172.17.0.9:27018,172.17.0.10:27018,172.17.0.11:27018/test?replicaSet=rs_shardsvr0
```

```
mongodb://172.17.0.12:27018,172.17.0.13:27018,172.17.0.14:27018/test?replicaSet=rs_shardsvr1
```

## 三、创建集群入口并关联配置集

**默认是mongod(分片处理shard模式)**，我们需要将起修改为**mongos(路由模式)**，负责路由和协调操作，使得集群像一个整体的系统

```
docker run --name mongos0 -d -p 27017:27017 --entrypoint "mongos" mongo --configdb rs_configsvr/172.17.0.3:27019,172.17.0.6:27019,172.17.0.8:27019 --bind_ip_all
```

通过**-p 27017:27017**映射方便访问

## 四、在集群入口(路由)上挂载分片集

```
docker exec -it mongos0 bash
mongo --host 172.17.0.7 --port 27017
sh.addShard("rs_shardsvr0/172.17.0.9:27018,172.17.0.10:27018,172.17.0.11:27018")
sh.addShard("rs_shardsvr1/172.17.0.12:27018,172.17.0.13:27018,172.17.0.14:27018")
```

输出结果，配置成功：

```
mongos> sh.addShard("rs_shardsvr0/172.17.0.9:27018,172.17.0.10:27018,172.17.0.11:27018")
{
        "shardAdded" : "rs_shardsvr0",
        "ok" : 1,
        "operationTime" : Timestamp(1584168954, 7),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584168954, 7),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
mongos> sh.addShard("rs_shardsvr1/172.17.0.12:27018,172.17.0.13:27018,172.17.0.14:27018")
{
        "shardAdded" : "rs_shardsvr1",
        "ok" : 1,
        "operationTime" : Timestamp(1584168957, 6),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584168957, 6),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
```

## 五、测试

进入路由容器创建test数据库并启用分片，分片 Collection对 test.order 的 _id 字段进行哈希分片

```
sh.enableSharding("test")
sh.shardCollection("test.order", {"_id": "hashed" })
```

```
{
        "ok" : 1,
        "operationTime" : Timestamp(1584169082, 5),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584169082, 5),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
{
        "collectionsharded" : "test.order",
        "collectionUUID" : UUID("566dec75-2585-4174-a8c5-32add9d7dfde"),
        "ok" : 1,
        "operationTime" : Timestamp(1584169146, 31),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584169146, 31),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
```

插入10000+条数据测试

```
for (i = 1; i <= 10000; i=i+1){
    db.order.insert({'price': i})
}
......
```

```
mongos> db.order.find().count()
10536
```

切换到分片查看数据统计个数：

```
rs_shardsvr0:PRIMARY> db.order.count()
5222
```

```
db.order.count()
5314
```

大功告成！

## 六、切分的优点

### 读/写

MongoDB将读写工作负载分发到[碎片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)在[切分簇](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)，允许每个碎片处理集群操作的子集。通过添加更多的碎片，可以在集群中水平地缩放读写工作负载。

对于包含碎片键或[复配](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)碎片钥匙[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以将查询针对特定的碎片或一组碎片。这些[目标行动](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted)通常比[广播](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)集群中的每一个碎片。

### 存储容量

[切分](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)将数据分发到[碎片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)在集群中，允许每个碎片包含整个集群数据的子集。随着数据集的增长，附加的碎片会增加集群的存储容量。

### 高可用性

A [切分簇](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)可以继续执行部分读/写操作，即使一个或多个碎片不可用。虽然在停机期间无法访问不可用碎片上的数据子集，但针对可用碎片的读或写仍然可以成功。

从MongoDB3.2开始，您可以部署[配置服务器](https://docs.mongodb.com/manual/reference/glossary/#term-config-server)如[复制集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)。只要大多数副本集可用，具有Config Server副本集(CSRS)的分片集群可以继续处理读和写。