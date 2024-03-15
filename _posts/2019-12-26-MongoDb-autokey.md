---
title: MongoDB主键自增
date:  	2019-12-26 19:56:36 +0800
category: Reference
tags: [Java,MongoDB]
excerpt: MongoDB在Java项目上实现autokey主键自增
---

### 前言

​	MongoDB的文档固定是使用“_id”作为主键的，它可以是任何类型的，默认是个ObjectId对象（在Java中则表现为字符串），那么为什么MongoDB没有采用其他比较常规的做法（比如MySql的自增主键），而是采用了ObjectId的形式来实现，主要是ObjectId采用时间戳+机器标识符+PID+计数器来实现的。

​	因此，MongoDB不使用自增主键，而是使用ObjectId。在分布式环境中，多个机器同步一个自增ID不但费时且费力，MongoDB从一开始就是设计用来做分布式数据库的，处理多个节点是一个核心要求，而ObjectId在分片环境中要容易生成的多。

### autokey主键自增的意义

​	前言可见ObjectId有很多的好处，可是在某种场景下，例如我在学校图书馆做的项目中，既要利用到mongodb存储大量的学生数据，又要记录某位学生是第几个使用该功能时，设置主键自增就很好地实现这个需求。

### 定义注解AutoIncKey

```
//annotation
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoIncKey {

}
```

### 定义序列实体类SeqInfo

```
//entity
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;
//id自动增长

@Document(collection = "sequence")
public class SeqInfo {
    @Id
    private String id;// 主键

    @Field
    private String collName;// 集合名称

    @Field
    private Long seqId;// 序列值

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getCollName() {
        return collName;
    }

    public void setCollName(String collName) {
        this.collName = collName;
    }

    public Long getSeqId() {
        return seqId;
    }

    public void setSeqId(Long seqId) {
        this.seqId = seqId;
    }
}
```

### 定义监听类SaveEventListener

```
//listener
package biyeseason.autokey.listener;

import com.example.weixin.autokey.annotation.AutoIncKey;
import com.example.weixin.autokey.entity.SeqInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.FindAndModifyOptions;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.mapping.event.AbstractMongoEventListener;
import org.springframework.data.mongodb.core.mapping.event.BeforeConvertEvent;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Component;
import org.springframework.util.ReflectionUtils;

import java.lang.reflect.Field;


@Component
public class SaveEventListener extends AbstractMongoEventListener<Object> {

    @Autowired
    private MongoTemplate mongo;

    @Override
    public void onBeforeConvert(BeforeConvertEvent<Object> event) {
        final Object source = event.getSource();
        if (source != null) {
            ReflectionUtils.doWithFields(source.getClass(), new ReflectionUtils.FieldCallback() {
                public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
                    ReflectionUtils.makeAccessible(field);
                    // 如果字段添加了我们自定义的AutoIncKey注解
                    if (field.isAnnotationPresent(AutoIncKey.class)) {
                        // 设置自增ID
                        field.set(source, getNextId(source.getClass().getSimpleName()));
                    }
                }
            });
        }
    }

    /**
     * 获取下一个自增ID
     *
     * @param collName
     *            集合（这里用类名，就唯一性来说最好还是存放长类名）名称
     * @return 序列值
     */
    private Long getNextId(String collName) {
        Query query = new Query(Criteria.where("collName").is(collName));
        Update update = new Update();
        update.inc("seqId", 1);
        FindAndModifyOptions options = new FindAndModifyOptions();
        options.upsert(true);
        options.returnNew(true);
        SeqInfo seq = mongo.findAndModify(query, update, options, SeqInfo.class);
        return seq.getSeqId();
    }
}
```

### 定义业务实体类User

```
import biyeseason.autokey.annotation.AutoIncKey;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

@Document(collation = "User")
public class User {
    @AutoIncKey
    @Id
    private Long id = 0L;

    @Field
    String uid;//用户ID

    @Field
    String pid;//图片ID

    @Field
    String sno;//学号

    @Field
    String name;//姓名

    @Field
    String dept;//专业

    @Field
    String openid;

    @Field
    String phoneNum;//手机号码

    @Field
    String create;//注册时间

    @Field
    String lastlogin;//最后登陆时间
    
    //Getter and Setter
```

### 最后实现

```
public void save() {
	User user = new User();
	//设置User信息
	service.save(user);
	// service.update(user);
	System.out.println("已生成ID：" + stu.getId());
}
```

