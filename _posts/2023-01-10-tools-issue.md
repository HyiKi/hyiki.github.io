---
title: 常用工具类的issue
date:  2023-01-10 12:11:03 +0800
category: Original
tags: SQL
excerpt: 记录日常遇到的工具类问题
---

1. [readTree can't parse an xml with a collection of elements with the same tag (UPDATE can in 2.12+)](https://github.com/FasterXML/jackson-dataformat-xml/issues/187)

   - 问题dependency：
     ```xml
     <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-core</artifactId>
       <version>2.11.3</version>
     </dependency>
     ```

   - issue背景：在对接物流商的轨迹接口，用的xml列表进行数据交互，结果多个元素的列表只解析出来一个

   - issue描述：版本在2.12之前，xml解析相同tag的列表时，只会解析出一个元素

   - issue举例：断言长度为2的列表，结果断言失败，实际上解析出来的长度只有1
     ```java
     String xmlStr = "<bla><statusFlags>\n" +
                 "                <statusFlagName>PWORD</statusFlagName>\n" +
                 "                <statusFlagName>NO_DEPLIM</statusFlagName>\n" +
                 "            </statusFlags></bla>";
     Bla resultObj = JacksonUtils.mapper.readValue(xmlStr, Bla.class);
     
     Assert.assertEquals(2, resultObj.someArray.size());
     ```

   - issue解决：升级版本2.12+，【注意】由于spring-boot套件包含jackson，建议版本跟随spring-boot版本
     ```xml
     <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-core</artifactId>
       <version>2.12.6</version>
     </dependency>
     ```

   - appendix：plugins安装Maven Helper，分析项目依赖

2. [数据解析小数精度丢失异常](https://github.com/alibaba/easyexcel/issues/1089)

   - 问题dependency：

     ```xml
     <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>easyexcel</artifactId>
       <version>2.2.7</version>
     </dependency>
     ```

   - issue背景：业务通过Excel导入数据，数据持久化进入系统时发现精度丢失

   - issue描述：easyexcel解析小数时，可能会出现精度丢失

   - issue举例：Excel表格里的0.625会被解析成0.6249999999999999，用String、Bigdecimal接收都一样

   - issue解决：升级版本

     ```xml
     <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>easyexcel</artifactId>
       <version>2.2.10</version>
     </dependency>
     ```

3. [gson将Integer默认转换成Double的问题](https://github.com/google/gson/issues/1084)

   - 问题dependency：2.8.8之前的版本
     ```xml
     <dependency>
         <groupId>com.google.code.gson</groupId>
         <artifactId>gson</artifactId>
         <version>2.8.8</version>
     </dependency>
     ```

   - issue背景：在进行json转Map的转换，解析query参数

   - issue描述：fromJson转Map方法，Number类型的数据默认转换为Double型

   - issue举例：整数0会转换为小数0.0

   - issue解决：升级版本至2.8.9+版本的Gson
     ```xml
     <dependency>
         <groupId>com.google.code.gson</groupId>
         <artifactId>gson</artifactId>
         <version>2.8.9</version>
     </dependency>
     ```

     

