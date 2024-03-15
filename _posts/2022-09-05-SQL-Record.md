---
title: SQL 记事本
date:  2022-09-05 19:11:03 +0800
category: Original
tags: SQL
excerpt: 记录会想不起来的SQL用法
---

[TOC]

---

### 1 offset - 偏移量

```sql
SELECT DISTINCT Salary
    FROM Employee 
    ORDER BY Salary DESC
    LIMIT 1
    OFFSET 1
```

### 2 IFNULL - 判空，类似getOrDefault用法

```
IFNULL(expression,alt_value)
```

### 3 SET - 存储过程设值

```
BEGIN
    SET N := N-1;
```

### 4 @ - 查询变量赋值

```
@age:=18
```

### 5 rank - 排名系列函数

```mysql
RANK() OVER(ORDER BY XX DESC) # 并列会占用名次
DENSE_RANK() OVER(ORDER BY XX DESC) # 并列不占用名次
ROW_NUMBER() # 递增
NTILE(n) over(ORDER BY XX DESC) # 按顺序平均分组排名
```

### 6 去重筛选第一条

```sql
-- with 写法
(with T1 as select MAX(唯一标识id),去重字段 where 条件 group by 去重字段) 再用唯一标识id去查到那条记录
-- 子查询 多行记录查询取一条,拿到想要的id直接 join 联表查询
select logistics_number, max(id) from ... group by logistics_number
```

### 7 分析SQL

```sql
-- 不执行，执行引擎预测
explain [SQL]
-- 实际执行，分析更准
explain analyze [SQL]
-- 实际执行，分析缓存命中：hit 命中缓存，read 读磁盘
explain (ANALYSE,BUFFERS)
```

### 8 索引使用情况

```sql
-- 查询使用情况
select * from pg_stat_all_indexes where schemaname = 'schema' and relname = 'table';
```

### 9 查询分片

```sql
<if test="isShard == true">
    and mod(lt.id, #{shardTotal}) = #{shardIndex}
</if>
```

### 10 Insert Conflict Update Return

```sql
-- conflict 里的属性必须是一组唯一索引（否则出错）
-- mybatis 返回值
<insert id="enableTrackConfig" useGeneratedKeys="true" keyProperty="id" keyColumn="id">
        insert into config(config_table,config_id,update_time)
        values(#{configTable}, #{configId},now())
        ON CONFLICT (config_id,config_table) DO
        UPDATE
        <trim prefix="set" suffixOverrides=",">
            update_time = now(),
        </trim>
        returning id
</insert>
```

### 11 PG数据库配置查询

```sql
-- 查询当前连接数
select count(*), usename
from pg_stat_activity
group by usename;
-- 查询最大连接数
show max_connections;
-- 查询数据库事务等级
show default_transaction_isolation;
-- 设置事务隔离级别
set default_transaction_isolation = 'read committed';
-- 查看数据库版本
select version();
```

### 12 COUNT（条件）

```sql
// 为什么要加上 OR NULL ? 因为SQL的count(NULL) 是不计数的，而count(false) 是会计数的
select count(tmd.*hyiki*_code = 1 OR NULL)
// ...
```

### 13 SUM （条件）

```sql
sum(case wr.receipt_status when 4 then wrs.batch_num else 0 end)
```

---

### 14 ERROR: canceling statement due to conflict with recovery

发生此错误的原因可能是主实例对只读副本上发生的活动缺乏可见性。无法将 WAL 信息应用于只读副本时，就会发生恢复冲突，因为这些更改可能会阻碍只读副本上正在发生的活动。

例如，假设在主实例中已删除表的只读副本上执行长时间运行的 SELECT 语句时，您在主实例上运行了 DROP 语句。然后，只读副本有两个选项：

- 等待 SELECT 语句完成后再应用 WAL 记录。在此情况下，复制滞后会增加。
- 应用 WAL 记录，然后取消 SELECT 语句。在此情况下，您会收到错误“**由于与恢复冲突而取消语句**”。

只读副本根据参数 **[max_standby_streaming_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-STREAMING-DELAY)** 和 [**max_standby_archive_delay**](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-ARCHIVE-DELAY) 的值来解决这些复制冲突。**max_standby_streaming_delay** 参数确定只读副本在取消与即将应用的 WAL 条目冲突的备用查询之前必须等待多长时间。如果冲突语句在此时间段之后仍在运行，则 PostgreSQL 将取消该语句并发出以下错误消息：

```plainText
ERROR: canceling statement due to conflict with recovery
```

出现此错误通常是由于在只读副本上长时间运行查询。

在前面运行 DROP 语句的示例中，DROP 请求存储在 WAL 文件中，以便稍后在只读副本上应用以保持一致性。假设只读副本上已经运行一条 SELECT 语句，该语句尝试从已删除的对象中检索数据，其运行时间大于 max_standby_streaming_delay 中的值。然后，将取消 SELECT 语句，以便应用 DROP 语句。

**会话 1**（只读副本）在 **example_table** 上运行 SELECT语句：

```plainText
postgres=> SELECT * from example_table;
```

**会话 2**（主实例）在 **example_table** 上运行 DROP 语句：

```plainText
postgres=> DROP TABLE example_table;
```

您收到以下错误：

```plainText
postgres@postgres:[27544]:ERROR:  canceling statement due to conflict with recovery
postgres@postgres:[27544]:DETAIL:  User was holding a relation lock for too long.
postgres@postgres:[27544]:STATEMENT:  select * from example_table;
```

此外，当只读副本上的事务正在读取主实例上设置为删除的元组时，可能会发生查询冲突。删除元组，然后在主实例上执行清理操作，这会导致与仍在副本上运行的 SELECT 查询发生冲突。在此情况下，副本上的 SELECT 查询将终止，并显示以下错误消息：

```plainText
ERROR:  canceling statement due to conflict with recovery
DETAIL: User query might have needed to see row versions that must be removed.
```

#### 解决方案

当只读副本遇到冲突，并且在错误日志中收到“由于与恢复冲突而导致取消语句”错误时，您可以根据错误消息设置某些自定义参数，以减少冲突的影响。请注意，必须在只读副本上设置自定义参数。

##### 您会收到错误“由于与恢复冲突而取消语句”和“详细信息：用户持有关系锁的时间过长”

[max_standby_streaming_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-STREAMING-DELAY)/[max_standby_archive_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-ARCHIVE-DELAY)：您可以使用这些参数在取消与即将应用的 WAL 条目冲突的备用语句之前留出更多时间。这些值表示从主实例收到数据后允许应用 WAL 数据的总时间。这些参数的指定取决于从何处读取 WAL 数据。如果从流式传输复制中读取 WAL 数据，则使用 max_standby_streaming_delay 参数。如果从 Amazon Simple Storage Service (Amazon S3) 中的存档位置读取 WAL 数据，则使用 max_standby_archive_delay 参数。

设置这些参数时，请记住以下几点：

- 如果将这些参数的值设置为 -1，则允许副本实例永远等待冲突查询完成，从而增加复制滞后。
- 如果将这些参数的值设置为 0，则会取消冲突的查询，并将 WAL 条目应用于副本实例。
- 这些参数的默认值设置为 30 秒。
- 如果在设置这些参数时未指定单位，则以毫秒为单位。

根据您的使用案例调整这些参数的值，以平衡查询取消或复制滞后。

**注意：**如果正在增加 [max_standby_archive_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-ARCHIVE-DELAY) 以避免取消与读取 WAL 存档条目冲突的查询，那么也要考虑增加 [max_standby_streaming_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-STREAMING-DELAY) 以避免与流式传输 WAL 条目冲突相关的取消。

##### 您会收到错误“由于与恢复冲突而取消语句”和“详细信息：查看必须删除的行版本可能需要执行用户查询”

[hot_standby_feedback](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-HOT-STANDBY-FEEDBACK)：如果激活此参数，则会从只读副本向主实例发送反馈消息，其中包含最早的活动事务的信息。因此，主实例不会删除事务可能需要的记录。

当您在只读副本上激活此参数时，长时间运行的只读副本查询可能会导致主实例上的表膨胀。这是因为清理操作不会移除只读副本上运行的查询可能需要的死元组。默认情况下，此参数处于关闭状态。因此，激活此参数时务必小心。

您还可以检查只读副本上的 [pg_stat_database_conflicts](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-CONFLICTS-VIEW) 视图，了解由于与只读副本上恢复冲突而取消的语句的统计信息。

---

### 15 seq

什么是字段的序列，序列常用于id的自增，按照1,2,3...的顺序依次填入每行，如果出现脏数据占用了后面序列的id会出现以下错误：

以下面的报错举例，下一个序列号id是1702，但是在执行自增之前，某个宝崽手动插入了一条id也是1702的数据，导致在执行自增的时候因为唯一约束报错

```bash
[2022-12-01 12:00:16] [23505] ERROR: duplicate key value violates unique constraint "logistics_freight_config_pk"
[2022-12-01 12:00:16] 详细：Key (id)=(1702) already exists.
```

如何解决？

```sql
-- 查出序列标识为logistics_freight_config_id_seq的值
-- 结果为 1702，说明下一个序列值是1702
SELECT nextval('plutus.logistics_freight_config_id_seq');
-- 设置下一个自增序列值，比如这里设置成1800，解决脏数据的冲突，完事~
select setval('plutus.logistics_freight_config_id_seq',1800);
```

---

### 16 update join

修数据遇到的一种SQL用法，事情是这样的：微服务架构中使用状态乐观锁机制update数据，update失败会出现这种情况，一张表的数据更新没有同步到另外一张表，发生这种问题往往采用定时任务进行监控，如果这些数据需要人工介入，这时候update join的作用就发挥出来了，使用表关联来实现数据的修复

下面示例是在POSTGRESQL通过关联表来update发货时间，了解到MySQL的语法会有所不同，程序员需要根据实际的数据库类型来使用不同的语法

```sql
-- POSTGRESQL
update logistics_tracking
set delivery_time = p.delivery_time
from logistics_tracking lt
         left join package p on lt.package_id = p.id
where logistics_tracking.logistics_number = 'ABC'
  and logistics_tracking.logistics_number = lt.logistics_number;
```

---

### 17 数组包含 any (ARRAY)

```sql
select count(1) from "order" where 'Bazar' = any (tags)
```

---

### 18 从阿里云日志搜索服务学 MySQL

服务在日常运行过程中，发生线上异常我们的第一反应是去看线上日志，首先这要求我们开发过程中要养成在业务操作上打印有效的日志，其次我们还要具备从日志信息中排查线上异常的能力，这次我要介绍的是从阿里云日志中快速查出关键信息的小技巧：

背景是物流系统对接了物流商的第三方接口，在获取面单数据为空时会打印日志：
`6 --- [     waybill-executor-thread-8] c.*hyiki*.hades.infr.logistics.gateway.logisticsStrategy.AbstractLogisticsService.uploadLabel()-591 : <0.263><1611756197029539840> 500730074622:物流商响应面单数据为空`

需求是捞出异常面单的运单号以及异常时间（精确到秒）

1. 使用【物流商响应面单数据为空*】查出目标日志

2. 使用管道拼接SQL语句加工日志【物流商响应面单数据为空* | SQL】

3. 转换成从SQL语句入手 加工出想要的数据

   - 异常面单的运单号：regexp_extract(regexp_extract(Message, '\w+:物流商响应面单数据为空'), '\w+')
     regexp_extract(x, regular expression[,  groupid])	提取并返回目标字符串中符合正则表达式的第[groupid]个子串。
     
     - 示例日志
       ```txt
       content: 2023-09-28 11:04:05.949  INFO  10 [TID:N/A]  --- [istics_event_16] .s.p.a.h.c.m.HadesLogisticsEventListener : HadesLogisticsEventListener consume success, messageId: AC0106F000064CF1BD4A8D6BB403A506, tag: LOGISTICS_CREATED_EVENT_TAG, time: 55 ms
       ```
     
     - 查询语句
     
       括号分组，groupId/index 指定第几个，0代表匹配所有
     
       ```sql
       SELECT regexp_extract(content , '(tag:) (\w+)', 2) as tag
       ```
     
     - 结果
     
       ```txt
       LOGISTICS_CREATED_EVENT_TAG
       ```
     
     - 场景
     
       使用日志统计每个消息tag的平均耗时
     
       ```sql
       HadesLogisticsEventListener and success | SELECT regexp_extract(content , '(tag:) (\w+)', 2)  as tag  ,avg( CAST((regexp_extract(content, '(time:) (\d+) ms', 2)) as bigint)) as time group by regexp_extract(content , '(tag:) (\w+)', 2)
       ```
     
       结果表格
     
       | tag                                 | time        |
       | ----------------------------------- | ----------- |
       | LOGISTICS_ABNORMAL_PUSH_EVENT_TAG   | 11.77324376 |
       | LOGISTICS_ESTIMATE_TIME_CONF_UPDATE | 9.666666667 |
       | LOGISTICS_ESTIMATE_FEE_SAVING_TAG   | 8.005588674 |
       | LOGISTICS_NUMBER_UPDATE_TAG         | 36.5367062  |
       | LOGISTICS_EXCHANGED_EVENT           | 13.98362573 |
       | LOGISTICS_GET_LABEL_SUCCESS_TAG     | 2.416902548 |
       | LOGISTICS_CREATED_EVENT_TAG         | 31.8422184  |
       | LOGISTICS_BATCH_SIGNED_TAG          | 9.569359331 |
       | LOGISTICS_OUTSIDE_EVENT_TAG         | 10.06065443 |
       | LOGISTICS_COD_STATUS_MODIFIED_EVENT | 7.25        |
       | EXCEL_EXPORT_TAG                    | 9.25        |
       | LOAD_CONTAINER_EVENT                | 1.827862874 |
     
   - 异常时间（精确到秒）：date_trunc('second', __time__)

   ```sql
   物流商响应面单数据为空* | SELECT date_trunc('second', __time__) as time, regexp_extract(regexp_extract(Message, '\w+:物流商响应面单数据为空'), '\w+') as logistics_number
   ```

4. 查出来的结果发现 重试获取面单会有重复的物流号 因此需要对结果集去重（同一个物流号取最早时间）
   ```sql
   物流商响应面单数据为空* | SELECT min(date_trunc('second', __time__)) as time, regexp_extract(regexp_extract(Message, '\w+:物流商响应面单数据为空'), '\w+') as logistics_number group by logistics_number
   ```

5. 完成，查出来部分结果集如下

   | time                    | logistics_number |
   | ----------------------- | ---------------- |
   | 2023-01-08 02:06:47.000 | 500730074994     |
   | 2023-01-08 00:06:44.000 | 500730074622     |
   | 2023-01-08 02:10:17.000 | 500730075020     |

---

### 19 JSON 函数和操作符

​	在查询轨迹内容时有这么一个需求："帮我查出节点码是600的时间"，然而轨迹在数据库里存储的格式是jsonb，于是需要用到SQL的json函数操作：http://www.postgres.cn/docs/9.4/functions-json.html

1. 原jsonb形式的轨迹数据
   ```sql
   select tracking_desc from logistics_tracking where logistics_number = 'xxx';
   ```

   ```json
   {
   	"item":[
   		{
   			"courierTime":"2023-01-06 17:13:05",
   			"processLocation":"",
   			"*hyiki*Code":"900",
   			"courierDesc":"return requested",
   			"courierCode":"PINSHENG_ND"
   		},
   		{
   			"courierTime":"2023-01-06 11:59:40",
   			"processLocation":"",
   			"*hyiki*Code":"650",
   			"courierDesc":"in transit to next facility in sydney west nsw",
   			"courierCode":"PINSHENG_ND"
   		},
   		{
   			"courierTime":"2023-01-06 11:49:12",
   			"processLocation":"melbourne airport vic",
   			"*hyiki*Code":"600",
   			"courierDesc":"item processed at facility",
   			"courierCode":"PINSHENG_ND"
   		},
   		{
   			"courierTime":"2023-01-05 10:37:53",
   			"processLocation":"airport west vic",
   			"*hyiki*Code":"600",
   			"courierDesc":"received by our network",
   			"courierCode":"PINSHENG_ND"
   		}
   	]
   }
   ```

2. 分割轨迹中的item数组
   ```sql
   select jsonb_array_elements(tracking_desc -> 'item') item
       from logistics_tracking
       where logistics_number = 'xxx';
   ```

   ```csv
   "{""*hyiki*Code"": ""900"", ""courierCode"": ""PINSHENG_ND"", ""courierDesc"": ""return requested"", ""courierTime"": ""2023-01-06 17:13:05"", ""processLocation"": """"}"
   "{""*hyiki*Code"": ""650"", ""courierCode"": ""PINSHENG_ND"", ""courierDesc"": ""in transit to next facility in sydney west nsw"", ""courierTime"": ""2023-01-06 11:59:40"", ""processLocation"": """"}"
   "{""*hyiki*Code"": ""600"", ""courierCode"": ""PINSHENG_ND"", ""courierDesc"": ""item processed at facility"", ""courierTime"": ""2023-01-06 11:49:12"", ""processLocation"": ""melbourne airport vic""}"
   "{""*hyiki*Code"": ""600"", ""courierCode"": ""PINSHENG_ND"", ""courierDesc"": ""received by our network"", ""courierTime"": ""2023-01-05 10:37:53"", ""processLocation"": ""airport west vic""}"
   ```

3. 将json数组里面的各项转成一张新表

   ```sql
   select td.item ->> '*hyiki*Code'       *hyiki*_code,
          td.item ->> 'courierCode'     courier_code,
          td.item ->> 'courierDesc'     courier_desc,
          td.item ->> 'courierTime'     courier_time,
          td.item ->> 'processLocation' process_location
   from (select jsonb_array_elements(tracking_desc -> 'item') item
         from logistics_tracking
         where logistics_number = 'xxx') td;
   ```

   ```csv
   900,PINSHENG_ND,return requested,2023-01-06 17:13:05,""
   650,PINSHENG_ND,in transit to next facility in sydney west nsw,2023-01-06 11:59:40,""
   600,PINSHENG_ND,item processed at facility,2023-01-06 11:49:12,melbourne airport vic
   600,PINSHENG_ND,received by our network,2023-01-05 10:37:53,airport west vic
   ```

4. 将这张新表命名为T1，这样就可以对轨迹进行数据处理了
   ```sql
   with T1 as (select td.item ->> '*hyiki*Code'       *hyiki*_code,
                      td.item ->> 'courierCode'     courier_code,
                      td.item ->> 'courierDesc'     courier_desc,
                      td.item ->> 'courierTime'     courier_time,
                      td.item ->> 'processLocation' process_location
               from (select jsonb_array_elements(tracking_desc -> 'item') item
                     from logistics_tracking
                     where logistics_number = 'xxx') td)
   select *
   from T1
   where T1.*hyiki*_code = '600';
   ```

   ```csv
   600,PINSHENG_ND,item processed at facility,2023-01-06 11:49:12,melbourne airport vic
   600,PINSHENG_ND,received by our network,2023-01-05 10:37:53,airport west vic
   ```

5. 数据处理plus版本
   ```sql
   with T1 as (select td.logistic_number,
                      td.item ->> '*hyiki*Code'       *hyiki*_code,
                      td.item ->> 'courierCode'     courier_code,
                      td.item ->> 'courierDesc'     courier_desc,
                      td.item ->> 'courierTime'     courier_time,
                      td.item ->> 'processLocation' process_location
               from (select wr.logistic_number, jsonb_array_elements(tracking_desc -> 'item') item
                     from warehouse_return wr
                              left join logistics_tracking lt on wr.logistic_number = lt.logistics_number
                     where condition) td)
   select wr.logistic_number                                            "物流单号",
          min(wr.apply_time)                                            "申请退货时间节点",
          min(wr.delivery_time)                                         "已收货时间节点",
          min(T1.courier_time)                                          "最早节点时间",
          max(T1.courier_time)                                          "最晚节点时间",
          min(case when T1.*hyiki*_code = '600' then T1.courier_time end) "最早600节点时间",
          min(case when T1.*hyiki*_code = '800' then T1.courier_time end) "最早800节点时间"
   from warehouse_return wr
            left join T1 on T1.logistic_number = wr.logistic_number
   where condition
   group by wr.logistic_number;
   ```


---

### 20 实现INSERT OR UPDATE

从mybatisplus的SDK方法saveOrUpdate有感而发，实现类似insert into ... on confict (...) do ... 这段SQL的等价用法也能这么简单~

先看看代码是怎么写的：

```java
LambdaQueryWrapper<BusinessDataDO> eq = new QueryWrapper<BusinessDataDO>()
                .lambda()
                .eq(BusinessDataDO::getBusinessType, type)
                .eq(BusinessDataDO::getBusinessId, businessId)
                .eq(BusinessDataDO::getName, name);
saveOrUpdate(entity, eq);
```

再看看执行这段代码打印出来的日志：

```sh
2023-04-13 11:50:57.088  INFO 77892 --- [onPool-worker-3] com.shop*hyiki*.plutus.app.hades.consumer.LogisticsExchangedConsumer.consume()-48 : LOGISTICS_EXCHANGED_EVENT [{"eventTime":1681350000000,"logisticsNumber":"TT879949025GB","tag":"LOGISTICS_EXCHANGED_EVENT"}]
2023-04-13 11:50:59.271 DEBUG 77892 --- [onPool-worker-3] com.shop*hyiki*.plutus.infr.businessdata.mapper.BusinessDataDAO.update.debug()-137 : ==>  Preparing: UPDATE business_data SET business_type=?, business_id=?, name=?, value=? WHERE (business_type = ? AND business_id = ? AND name = ?)
2023-04-13 11:50:59.272 DEBUG 77892 --- [onPool-worker-3] com.shop*hyiki*.plutus.infr.businessdata.mapper.BusinessDataDAO.update.debug()-137 : ==> Parameters: logistics_number(String), TT879949025GB(String), exchanged_event_time(String), 2023-04-13 09:40:00(String), logistics_number(String), TT879949025GB(String), exchanged_event_time(String)
2023-04-13 11:50:59.324 DEBUG 77892 --- [onPool-worker-3] com.shop*hyiki*.plutus.infr.businessdata.mapper.BusinessDataDAO.update.debug()-137 : <==    Updates: 0
2023-04-13 11:50:59.326 DEBUG 77892 --- [onPool-worker-3] com.shop*hyiki*.plutus.infr.businessdata.mapper.BusinessDataDAO.insert.debug()-137 : ==>  Preparing: INSERT INTO business_data ( business_type, business_id, name, value ) VALUES ( ?, ?, ?, ? )
2023-04-13 11:50:59.326 DEBUG 77892 --- [onPool-worker-3] com.shop*hyiki*.plutus.infr.businessdata.mapper.BusinessDataDAO.insert.debug()-137 : ==> Parameters: logistics_number(String), TT879949025GB(String), exchanged_event_time(String), 2023-04-13 09:40:00(String)
```

分析：
在同一个事务里，先按照指定的查询条件更新，如果更新的条数为0，再执行插入entity的动作。

---

### 21 使用INSERT ON CONFLICT覆盖写入数据 & 伪表excluded

- 前提：初始化一张测试数据表
  ```sql
  CREATE TABLE t1
  (
      a int PRIMARY KEY,
      b int,
      c int,
      d int DEFAULT 0
  );
  ```

- 众所周知，INSERT语句支持多行语句的插入，UPDATE语句支持单行语句的更新。语法如下：

  - INSERT
    ```sql
    INSERT INTO t1
    VALUES (0, 0, 0, 0)
    ```

  - UPDATE
    ```sql
    UPDATE t1
    SET b = 1,
        c = 2,
        d = 3
    WHERE a = 0;
    ```

- 那么在使用postgresql的INSERT ON CONFLICT语法，怎么实现多行语句的 INSERT ON CONFLICT DO UPDATE，这里引入“伪表”的概念

  - 伪表

    ```sql
    INSERT INTO t1 VALUES (2,2,2,2) ON CONFLICT (a) DO UPDATE SET (b, c, d) = (excluded.b, excluded.c, excluded.d);
    ```

    在DO UPDATE SET子句中，可以使用excluded表示冲突的数据构成的伪表，在主键冲突的情况下，引用伪表中列的值覆盖原来列的值。上述语句中，新插入的数据`(0,2,2,2)`构成了一个伪表，伪表包含1行4列数据，表名为excluded，可以使用`excluded.b, excluded.c, excluded.d`去引用伪表中的列。

  - 满足多行写入的场景

    - 执行SQL前：

      | a    | b    | c    | d    |
      | :--- | :--- | :--- | :--- |
      | 0    | 0    | 0    | 0    |
      | 2    | 2    | 2    | 2    |

    - 执行SQL：
      ```sql
      INSERT INTO t1
      VALUES (0, 1, 1, 1),
             (1, 2, 2, 2),
             (2, 3, 3, 3)
      ON CONFLICT (a) DO UPDATE SET (b, c, d) = (excluded.b, excluded.c, excluded.d);
      ```

    - 执行SQL后：

      | a    | b    | c    | d    |
      | :--- | :--- | :--- | :--- |
      | 0    | 1    | 1    | 1    |
      | 1    | 2    | 2    | 2    |
      | 2    | 3    | 3    | 3    |

---

### 22 WITHIN GROUP

- **作用**：WITHIN GROUP 用于指定在聚合函数中如何处理排序和聚合值，通常与排序的聚合函数一起使用。

- **用法**：在 PostgreSQL 中，WITHIN GROUP 通常与排序的聚合函数一起使用，以指示在给定排序条件下如何计算聚合函数的结果。

- **示例**：计算中位数时，可以使用 PERCENTILE_CONT 函数，并使用 WITHIN GROUP 来指定排序条件。例如：

  ```sql
  SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY column_name) AS median
  FROM your_table;
  ```

---

### 23 ROW_NUMBER() OVER

- **作用**：ROW_NUMBER() OVER 是一种窗口函数，用于为结果集中的行分配唯一的数字序号。

- **用法**：ROW_NUMBER() OVER 可以根据指定的排序条件为每个分组中的行分配一个序号。

- **示例**：为每个部门中工资最高的员工分配排名。例如：

  ```sql
  SELECT
      employee_id,
      department_id,
      salary,
      ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank
  FROM employees;
  ```
