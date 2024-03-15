---
title: MongoTemplate基本操作
date:  	2020-02-12 12:56:36 +0800
category: Reference
tags: [Java,MongoDB]
excerpt: Java项目上操作MongoDB实现增删改查
---

### 导入依赖包

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```

### MongoConfig配置

在配置文件夹中，写MongoConfig配置

```
package com.example.weixin.config;

import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.MongoDbFactory;
import org.springframework.data.mongodb.core.convert.CustomConversions;
import org.springframework.data.mongodb.core.convert.DbRefResolver;
import org.springframework.data.mongodb.core.convert.DefaultDbRefResolver;
import org.springframework.data.mongodb.core.convert.DefaultMongoTypeMapper;
import org.springframework.data.mongodb.core.convert.MappingMongoConverter;
import org.springframework.data.mongodb.core.mapping.MongoMappingContext;

@Configuration
public class MongoConfig {
    @Bean
    public MappingMongoConverter mappingMongoConverter(MongoDbFactory factory, MongoMappingContext context, BeanFactory beanFactory) {
        DbRefResolver dbRefResolver = new DefaultDbRefResolver(factory);
        MappingMongoConverter mappingConverter = new MappingMongoConverter(dbRefResolver, context);
        try {
            mappingConverter.setCustomConversions(beanFactory.getBean(CustomConversions.class));
        }
        catch (NoSuchBeanDefinitionException ignore) {}

        // Don't save _class to mongo
        mappingConverter.setTypeMapper(new DefaultMongoTypeMapper(null));

        return mappingConverter;
    }
}
```

### Mongo连接配置文件

以.yml文件为例

```
data:
    mongodb:
      username: user_miniprogram
      password: ENC(NT3JIFFJpwJE2ziIn74GmoP5+5omqaV/Vpmns0X9c1U=)
      host: 172.19.13.51
      port: 27017
      database: library_miniprogram
      authentication-database: library_miniprogram
```

### MongoTemplate基本操作

在持久层中实现对MongoDB的基本操作

```
package com.wlsq.oauth.dao.impl;
 
import java.util.List;
 
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
 
import com.wlsq.oauth.dao.IUserDao;
import com.wlsq.oauth.dao.support.AbstractBaseMongoTemplete;
import com.wlsq.oauth.entity.User;
 
public class UserDao extends AbstractBaseMongoTemplete implements IUserDao {
	/**
	 * 新增
	 * <br>------------------------------<br>
	 * @param user
	 */ 
	public void insert(User user) {
		// TODO Auto-generated method stub
		mongoTemplate.insert(user);
	}
	/**
	 * 批量新增
	 * <br>------------------------------<br>
	 * @param users
	 */
	public void insertAll(List<User> users) {
		// TODO Auto-generated method stub
		mongoTemplate.insertAll(users);
	}
	/**
	 * 删除,按主键id, 如果主键的值为null,删除会失败
	 * <br>------------------------------<br>
	 * @param id
	 */
	public void deleteById(String id) {
		// TODO Auto-generated method stub
		User user = new User(id, null, 0);
		mongoTemplate.remove(user);
	}
	/**
	 * 按条件删除
	 * <br>------------------------------<br>
	 * @param criteriaUser
	 */
	public void delete(User criteriaUser) {
		// TODO Auto-generated method stub
		Criteria criteria = Criteria.where("age").gt(criteriaUser.getAge());;
		Query query = new Query(criteria);
		mongoTemplate.remove(query, User.class);
	}
	/**
	 * 删除全部
	 * <br>------------------------------<br>
	 */
	public void deleteAll() {
		// TODO Auto-generated method stub
		mongoTemplate.dropCollection(User.class);
	}
	/**
	 * 按主键修改,
	 * 如果文档中没有相关key 会新增 使用$set修改器
	 * <br>------------------------------<br>
	 * @param user
	 */
	public void updateById(User user) {
		// TODO Auto-generated method stub
		Criteria criteria = Criteria.where("id").is(user.getId());
		Query query = new Query(criteria);
		Update update = Update.update("age", user.getAge()).set("name", user.getName());
		mongoTemplate.updateFirst(query, update, User.class);
	}
	/**
	 * 修改多条
	 * <br>------------------------------<br>
	 * @param criteriaUser
	 * @param user
	 */
	public void update(User criteriaUser, User user) {
		// TODO Auto-generated method stub
		Criteria criteria = Criteria.where("age").gt(criteriaUser.getAge());;
		Query query = new Query(criteria);
		Update update = Update.update("name", user.getName()).set("age", user.getAge());
		mongoTemplate.updateMulti(query, update, User.class);
	}
	/**
	 * 根据主键查询
	 * <br>------------------------------<br>
	 * @param id
	 * @return
	 */
	public User findById(String id) {
		// TODO Auto-generated method stub
		return mongoTemplate.findById(id, User.class);
	}
	/**
	 * 查询全部
	 * <br>------------------------------<br>
	 * @return
	 */
	public List<User> findAll() {
		// TODO Auto-generated method stub
		return mongoTemplate.findAll(User.class);
	}
	/**
	 * 按条件查询, 分页
	 * <br>------------------------------<br>
	 * @param criteriaUser
	 * @param skip
	 * @param limit
	 * @return
	 */
	public List<User> find(User criteriaUser, int skip, int limit) {
		// TODO Auto-generated method stub
		Query query = getQuery(criteriaUser);
		query.skip(skip);
		query.limit(limit);
		return mongoTemplate.find(query, User.class);
	}
	/**
	 * 根据条件查询出来后 再去修改
	 * <br>------------------------------<br>
	 * @param criteriaUser  查询条件
	 * @param updateUser    修改的值对象
	 * @return
	 */
	public User findAndModify(User criteriaUser, User updateUser) {
		// TODO Auto-generated method stub
		Query query = getQuery(criteriaUser);
		Update update = Update.update("age", updateUser.getAge()).set("name", updateUser.getName());
		return mongoTemplate.findAndModify(query, update, User.class);
	}
	/**
	 * 查询出来后 删除
	 * <br>------------------------------<br>
	 * @param criteriaUser
	 * @return
	 */
	public User findAndRemove(User criteriaUser) {
		// TODO Auto-generated method stub
		Query query = getQuery(criteriaUser);
		return mongoTemplate.findAndRemove(query, User.class);
	}
	/**
	 * count
	 * <br>------------------------------<br>
	 * @param criteriaUser
	 * @return
	 */
	public long count(User criteriaUser) {
		// TODO Auto-generated method stub
		Query query = getQuery(criteriaUser);
		return mongoTemplate.count(query, User.class);
	}
	/**
	 *
	 * <br>------------------------------<br>
	 * @param criteriaUser
	 * @return
	 */
	private Query getQuery(User criteriaUser) {
		if (criteriaUser == null) {
			criteriaUser = new User();
		}
		Query query = new Query();
		if (criteriaUser.getId() != null) {
			Criteria criteria = Criteria.where("id").is(criteriaUser.getId());
			query.addCriteria(criteria);
		}
		if (criteriaUser.getAge() > 0) {
			Criteria criteria = Criteria.where("age").gt(criteriaUser.getAge());
			query.addCriteria(criteria);
		}
		if (criteriaUser.getName() != null) {
			Criteria criteria = Criteria.where("name").regex("^" + criteriaUser.getName());
			query.addCriteria(criteria);
		}
		return query;
	}
 
}
```

