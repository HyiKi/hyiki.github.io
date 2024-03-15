---
title: 常见注解的学习笔记
date:  	2020-02-22 21:56:36 +0800
category: Original
tags: [Java,Annotation,Learning]
excerpt: 用于长期可更新的常见注解的学习笔记
---

### 内置注解（JDK5开始）

- @SuppressWarning("unchecked")`压制警告`
- @Override`重载`
- @Deprecated`舍弃`

### 测试注解

- @RunWith(SpringRunner.class)

  @SpringBootTest

  - ```
    <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
              <exclusions>
                  <exclusion>
                      <groupId>org.junit.vintage</groupId>
                      <artifactId>junit-vintage-engine</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
    ```

- @BeforeClass`类之前`

- @Before`之前`

- @Test`测试`

- @After`之后`

- @AfterClass`类之后`

### @interface注解

- Retention(RetentionPolicy.生命周期)
- Target({ElementType.作用位置,……})

### Spring

- @RestController`控制层`
  - @Controller
  - @ResponseBody
- @GetMapping(url)
- @PostMapping(url)
- @Service`业务层`

### RestfulAPI

- @RestController `标明此Controller提供RestAPI`

- @RequestMapping及其变体`映射http请求url到java方法`

- @RequestParam `映射请求参数到java方法的参数`

  ```
  (@RequestParam(name="username",required=false,defaultValue="myName") String nickname
  ```

- @PageableDefault `指定分页参数默认值`

  ```
  @PageableDefault(page=1,size=10,sort="username,asc")
  //查询第一页，查询10条，按照用户名升序排列
  ```

- @PathVariable `映射url片段到java方法的参数`

  ```
  @RequestMapping(value="/user/{id}",method=RequestMethod.GET)
  public User getInfo(@PathVariable String id) {
  }
  ```

- 在url声明中使用正则表达式

  ```
  @RequestMapping(value="/user/{id:\\d+}",method=RequestMethod.GET)//如果希望对传递进来的参数作一些限制，使用正则表达式
  ```

- @JsonView `控制json输出内容`
  实体类设置：

  ```
  public interface UserSimpleView{};
  //有了该继承关系，在显示detail视图的时候同时会把simple视图的所有字段也显示出来
  public interface UserDetailView extends UserSimpleView{};
  ...
  @JsonView(UserSimpleView.class)
  public String getId() {
  	return id;
  }
  ...
  ```

  具体实现（控制层）：

  ```
  @GetMapping
  @JsonView(User.UserSimpleView.class)
  public List<User> query(@RequestParam(name="username",required=false,defaultValue="tom") String username){
      ...
      return user;
  }
  ```

### SpringIOC

- @Autowired`自动注入`
- @Bean

### SpringBoot

- @SpringBootConfiguration
- @EnableAutoConfiguration`可以智能的自动配置`
- @ComponentScan`做的事情就是告诉Spring从哪里找到bean`
- **@ConfigurationProperties和@EnableConfigurationProperties配合使用**
  - @ConfigurationProperties("")`读取配置文件指定Bean注入当前类`
    - @PostConstruct`在依赖注入完毕后再进行`
  - @EnableConfigurationProperties(类名.class)`将当前类放入IOC容器中` 

### 启动类

- @SpringBootApplication
- @MapperScan

### Mybatis注解

- @Select
  SQL语句
- @Results
  查询结果字段和Mapper字段不一致时，使用Results映射
  - @Result
    包含在Results的每个字段映射实现，id为主键，property为实体类中的字段，column为数据库中查询得到的字段
  - @Many
    用于联表操作实现一对多的关系，因此`javaType = List.class`需要包装成集合
  - @One
    用于联表操作实现一对一的关系，比如用教室id对于一个教室，那么`javaType = ClassRoom.class`包装

#### 实例

```
    @Select("select * from sys_user where username = #{username}")
    @Results({
            @Result(id = true, property = "id", column = "id"),
            @Result(property = "roles", column = "id", javaType = List.class,
                many = @Many(select = "com.itheima.mapper.RoleMapper.findByUid"))
    })
```

#### 依赖包

```
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
```

