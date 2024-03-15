---
title: 从Redis集群模式分片技术认识HashTag
date:  2022-07-08 16:04:11 +0800
category: Original
tags: Redis
excerpt: Redis的集群分片模式中，认识HashTag的重要性，怎么活用HashTag
---

# 从Redis集群模式分片技术认识HashTag

- 背景

  - 为什么需要分片
    - 首先在**单机模式下，不存在分片**一说(local redis default port 6379)
    - **主从模式 master-replica HA**（高可用主从热备模式，能够提供不错的服务)
      ![主从模式](/assets/img/redis/主从模式.png)
    - **哨兵模式 sentinel**（部署多一台哨兵，选举过程有保护机制，禁止写操作）![多哨兵模式](/assets/img/redis/多哨兵模式.png)
    - **以上模式都没有达到真正的数据 sharding 存储**，每个 Redis 实例中存储的都是全量数据
    - 而在**多机集群架构模式Redis cluster**下，当然是希望数据能够均匀地分布在每台集群的机子上，把每台机器的性能与存储空间都充分利用，同时在这我们不希望发生**数据倾斜**。
    
  - **集群分片的模式CLUSTER**
    
    - **客户端分片**：缺点
      **静态分片**，调整Redis实例数量的同时，需要运维人员手动调整分片的逻辑，提高运维人员的工作量；
      **维护成本大**，例如有多个项目同用Redis集群，那么一次改动要多次维护
      
    - 解决方案——**代理分片**
      已应用于Twitter、豌豆荚的代理分片，集中管理Redis分片，外部应用只管提供kv，由代理集群统一做分片处理，
      
      唯一的缺点在于通信过程需要损耗一些性能
      
      ![代理分片](/assets/img/redis/代理分片.png)

- 目的

  - 分析**Redis性能边界**，能够根据不同的业务体量，使用合适分布式缓存架构
    ![Redis性能边界](/assets/img/redis/Redis性能边界.png)
  - 场景1：存储资源消耗较低，使用**高可用主备架构**即可
    ![存储资源消耗较低](/assets/img/redis/存储资源消耗较低.png)
  - 场景2：数据量大，32G内存的机器占用了20G，Redis响应慢，而且使用 INFO 命令查看 latest_fork_usec 指标（RDB/最近一次 fork 耗时），发现特别高。推荐使用**Redis分布式多机集群模式**

- 过程

  - 分片算法：CRC算法

    ```
    循环冗余校验（英语：Cyclic redundancy check，通称“CRC”）是一种根据网络数据包或电脑文件等数据产生简短固定位数校验码的一种散列函数，主要用来检测或校验数据传输或者保存后可能出现的错误。生成的数字在传输或者存储之前计算出来并且附加到数据后面，然后接收方进行检验确定数据是否发生变化。由于本函数易于用二进制的电脑硬件使用、容易进行数学分析并且尤其善于检测传输通道干扰引起的错误，因此获得广泛应用。
    ```

    - 应用场景1：网络传输文件或数据时，使用CRC算法将其散列成固定长度的值拼接在传输的数据末端进行冗余，接收方根据接收到的数据进行**散列验证，判断数据在传输过程中是否受到干扰**。

    - 应用场景2：Redis多集群分片，将数据按照CRC算法的**计算值匹配对应Slot**，从而分配到不同的实例上。

    ![CLUSTER图示](/assets/img/redis/CLUSTER图示.png)

    - what's the difference between CRC-8 and CRC-16?
  
      ```
      CRC8 -> probability of failure is 1/256 (8bits)
      CRC16 -> probability of failure 1/65536 (16bits)
      ......
      ```

  - 分析hash示例源码，可知：
  
    - 未指定哈希标签，或者指定了空，会hash整个key
    - 指定哈希标签`{}`，会哈希第一个标签的key范围
    - 哈希值最终会取14位二进制，即取值范围 `2^14 = 16384` （0~16383）
    - Examples:
      - The two keys `{user1000}.following` and `{user1000}.followers` will hash to the same hash slot since only the substring `user1000` will be hashed in order to compute the hash slot.
      - For the key `foo{}{bar}` the whole key will be hashed as usually since the first occurrence of `{` is followed by `}` on the right without characters in the middle.
      - For the key `foo{{bar}}zap` the substring `{bar` will be hashed, because it is the substring between the first occurrence of `{` and the first occurrence of `}` on its right.
      - For the key `foo{bar}{zap}` the substring `bar` will be hashed, since the algorithm stops at the first valid or invalid (without bytes inside) match of `{` and `}`.
      - What follows from the algorithm is that if the key starts with `{}`, it is guaranteed to be hashed as a whole. This is useful when using binary data as key names.
    
      ```c
      unsigned int HASH_SLOT(char *key, int keylen) {
          int s, e; /* start-end indexes of { and } */
      
          /* Search the first occurrence of '{'. */
          for (s = 0; s < keylen; s++)
              if (key[s] == '{') break;
      
          /* No '{' ? Hash the whole key. This is the base case. */
          if (s == keylen) return crc16(key,keylen) & 16383;
      
          /* '{' found? Check if we have the corresponding '}'. */
          for (e = s+1; e < keylen; e++)
              if (key[e] == '}') break;
      
          /* No '}' or nothing between {} ? Hash the whole key. */
          if (e == keylen || e == s+1) return crc16(key,keylen) & 16383;
      
          /* If we are here there is both a { and a } on its right. Hash
           * what is in the middle between { and }. */
          return crc16(key+s+1,e-s-1) & 16383;
      }
      ```
    
  - 哈希标签该如何使用？遵循什么规则？
  
    - 同一对象的信息尽可能哈希到同一个槽中：保证事务能够在单个对象的范围发生
    
        | 钥匙           | 散列伪代码               | 哈希槽 |
        | -------------- | ------------------------ | ------ |
        | 用户资料：1234 | CRC16('1234') 模型 16384 | 6025   |
        | 用户会话：1234 | CRC16('1234') 模型 16384 | 6025   |
        | 用户资料：5678 | CRC16('5678') 模型 16384 | 3312   |
        | 用户会话：5678 | CRC16('5678') 模型 16384 | 3312   |
    
      这个正则表达式足够灵活，也可以处理以“user-”开头的其他键，所以我们可以有像“user-image”或“user-pagecount”这样的键。在这种方案中，每个用户的信息都将保存在单个哈希槽中，从而使各种**事务都可以在单个用户的范围内发生**。
    
    - 🌰举个例子：without hashslot there may be redirected to the other slot for the key insert into redis
        （不使用hashslot，这个key可能会被重定向到Redis的其他槽）
    
        - not using {user} tag for example is, use cluster keyslot command to show the value of crc16('key')
          （举个例子，不使用hashslot：同一个用户不同信息被分配到不同槽中）
    
          ```shell
          127.0.0.1:7000> sadd user:512:following user:271
          -> Redirected to slot [7578] located at 127.0.0.1:7001
          (integer) 1
          127.0.0.1:7001> cluster keyslot user:512:following
          (integer) 7578
          127.0.0.1:7001> sadd user:512:followed_by user:271
          -> Redirected to slot [3322] located at 127.0.0.1:7000
          (integer) 1
          127.0.0.1:7000> cluster keyslot user:512:followed_by
          (integer) 3322
          ```
    
        - this cause unspected error when I want to use SINTER to show the user who I follow and who followed by me,sth. wrong happen......
          （这会导致不符合预期的错误，当我想要用SINTER命令去展示当前用户的互相关注的有哪些用户的时候，错误发生了......）
    
          ```shell
          127.0.0.1:7000> sinter user:512:following user:512:followed_by
          (error) CROSSSLOT Keys in request don't hash to the same slot
          ```
    
        - that's mean commands will not work across charts they will only work within the same hash slot
          （那意味着命令将不会跨分片工作，只会在同一个槽中作用）
    
        - the correct way is as follows
          （正确的做法如下）
    
          ```shell
          127.0.0.1:7000> SADD user:{512}:following user:271
          (integer) 1
          127.0.0.1:7000> CLUSTER KEYSLOT user:{512}:following
          (integer) 3808
          127.0.0.1:7000> SADD user:{512}:followed_by user:271
          (integer) 1
          127.0.0.1:7000> CLUSTER KEYSLOT user:{512}:followed_by
          (integer) 3808
          127.0.0.1:7000> SINTER user:{512}:following user:{512}:followed_by
          1) "user:271"
          ```
  
- 总结

  1. 数据量不大的情况，使用**高可用主备架构**；数据量大，且影响备份性能，改用**集群模式CLUSTER**
  2. 巧用HashTag，底层逻辑是通过识别**第一个**`{}`里的内容，将其CRC16散列值进行分片
  3. 有**相同业务含义的数据应该分配到同一个分片**上，例如在单个对象范围内开启事务、在一行command的多个参数应存在于同一个分片（如集合操作）
