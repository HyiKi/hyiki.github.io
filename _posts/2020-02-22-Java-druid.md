---
title: Druid的配置使用
date:  	2020-02-22 14:56:36 +0800
category: Reference
tags: [Java,Developer_Tools]
excerpt: Spring中druid+mybatis连接池配置
---

### 导包

```
        <!--mybatis + druid连接池-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.20</version>
        </dependency>
        
        <!--MySQL数据库与Java连接 com.mysql.cj.jdbc.Driver-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

### 配置

在`application.properties`中配置，其中附带MySQL数据库与Java的连接

```
spring.datasource.druid.url= jdbc:mysql://url:port/gallery
spring.datasource.druid.username= username
spring.datasource.druid.password= password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jackson.time-zone=GMT+8
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
mybatis.type-aliases-package=com.hello.springboot.mapper

logging.level.linkgroup.dao=debug

#mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
mybatis.mapper-locations=classpath*:mapper/**/*.xml
spring.datasource.druid.initialSize=5
spring.datasource.druid.minIdle=5
spring.datasource.druid.maxActive=20
spring.datasource.druid.maxWait=60000
spring.datasource.druid.timeBetweenEvictionRunsMillis=60000
spring.datasource.druid.minEvictableIdleTimeMillis=300000
spring.datasource.druid.validationQuery=SELECT 1 FROM DUAL
spring.datasource.druid.testWhileIdle=true
spring.datasource.druid.testOnBorrow=false
spring.datasource.druid.testOnReturn=false
spring.datasource.druid.poolPreparedStatements=true
spring.datasource.druid.filters=stat,wall,log4j
spring.datasource.druid.maxPoolPreparedStatementPerConnectionSize=20
spring.datasource.druid.useGlobalDataSourceStat=true
spring.datasource.druid.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

### 解决

Java与MySQL数据库连接，以及连接池的应用。