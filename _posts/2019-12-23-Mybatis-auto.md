---
title: MyBatis自动配置
date:  	2019-12-23 08:56:36 +0800
category: Original
tags: [Developer_Tools,Java]
excerpt: IDEA利用MyBatis Plugins插件自动生成配置
---

### MyBatis-plus 安装

1. 依次进入`File`-`Setting`-`Plugins`
2. 点击`Browser Repositories`
3. 搜索`MyBatisX`并安装
4. 搜索`Free MyBatis plugin`并安装

### 修改pom文件

在`plugins`插件处插入

```
<plugins>
            <!-- mybatis generator 自动生成代码插件 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.5</version>
                <configuration>
                    <!--配置文件的位置-->
                    <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
            </plugin>
</plugins>
```

### 新建一个generatorConfig.xml

放入代码

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包-->
    <classPathEntry  location="D:\temp\mysql-connector-java-5.1.48-bin.jar"/>
    <context id="DB2Tables"  targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://120.24.171.53/gallery" userId="root" password="hiki2019">
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- 生成模型的包名和位置-->
        <javaModelGenerator targetPackage="com.example.mie.springboot.entity" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置-->
        <sqlMapGenerator targetPackage="main.resources.mapping" targetProject="src">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        <!-- 生成DAO的包名和位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.example.mie.springboot.dao" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名-->
        <table tableName="sc"
               domainObjectName="sc"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               selectByExampleQueryId="false">

        </table>

    </context>
</generatorConfiguration>
```

### generatorConfig.xml报错处理

"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd"报红报错

`file`–>`setting`–>`languages & frameworks`–>`Schemas and DTDs`->`Ignored Schemas and DTDs`

点击加号加入就可以了

### 下载数据库驱动包并更换路径

1. “https://dev.mysql.com/downloads/connector/j/”官网下载地址
2. 建议选择以前的驱动版本5.1.*
3. Select Operating System:Platform Independent
4. Download ->**Platform Independent (Architecture Independent), ZIP Archive**
5. 取出其中的`mysql-connector-java-5.1.48-bin.jar`作连接

### 运行

1. 单击配置，选择Edit Configurations
2. 点击加号，添加一项Maven
3. 在Command line 输入：`mybatis-generator:generate -e`
4. 应用确定后运行即可