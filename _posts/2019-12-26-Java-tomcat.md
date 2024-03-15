---
title: Java运行tomcat实例
date:  	2019-12-26 19:56:36 +0800
category: Original
tags: Java
excerpt: IDEA新建Java工程并放线上tomcat运行
---

### 开发环境

- IntelliJ IDEA 2018.2.4 x64

### New Project

1. `Spring Initializr`-Next
2. 选择所需的依赖在线生成

### 编辑pom.xml文件

##### 设置packaging打包文件格式

```
<packaging>war</packaging>
```

##### 设置打包名称

放在<build>标签下

```
<finalName>filename</finalName>
```

##### 置入tomcat依赖

放在<dependencies>标签下

```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
        <scope>provided</scope>
    </dependency>
	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
	</dependency>
```

### Application文件的重载

```
package biyeseason;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class BiyeseasonApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(BiyeseasonApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(BiyeseasonApplication.class);
    }
}
```

### Maven Projects打包上线

1. `Lifecycle`-`clean`清理已存在的target文件
2. `Lifecycle`-`package`打包目前工程到target文件
3. 在工程下的target文件中找到`war`文件并放入线上tomcat的`webapps`文件夹中