---
title: MongoDB 学习笔记
date:  	2020-03-13 08:56:36 +0800
category: Original
tags: [MongoDB,Learning]
excerpt: Document类型数据库，持续更新的学习笔记。
---

## 前言

MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

## Linux中下载并部署MongoDB

1. 使用Docker部署，对应的仓库https://hub.docker.com/_/mongo
2. `docker pull mongo`拉取线上仓库的MongoDB镜像
3. `docker run --name some-mongo -p 27017:27017 -d mongo:tag`将镜像变为容器，其中**some-mongo**为容器名，**tag**为需要创建的版本号
4. 通过`docker ps`就能看到创建的容器了
5. `docker exec -it mongodb bash`（默认SHELL)就可以进入我刚刚创建的mongodb容器了
6. 在容器中输入`mongo admin`以管理员身份进入MongoDB数据库
7. 下载辅助工具`NoSQLBooster for MongoDB`，https://www.nosqlbooster.com/，利用创建用户的操作得到认证的账号密码

### 创建用户

给admin数据库创建一个用户，方式如下：

```
db.createUser({user:"root",pwd:"123",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
```

user表示用户名，pwd表示密码，role表示角色，db表示这个用户应用在哪个数据库上。

**用户角色分类**

1. Read：允许用户读取指定数据库
2. readWrite：允许用户读写指定数据库
3. dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
4. userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
5. clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限
6. readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
7. readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
8. userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
9. dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限
10. root：只在admin数据库中可用。超级账号，超级权限

**使用角色登陆**

```
db.auth("root","123")
```

## MongoDB基本操作

### 客户端后台管理工具

1. 首先我们下载Robo 3T(下载地址https://robomongo.org/download)
2. 下载成功之后解压，找到.exe可执行文件双击启动
3. 启动后新建一个连接，输入ip地址
4. 连接成功之后，我们就可以看到数据库的信息了
5. 选中相应的数据库，右键`Open Shell`即可以对数据库进行操作

### 命令行

#### 基础操作

1. `db`查看当前所在的数据库
2. `show dbs`显示所有的数据库
3. `use <数据库>`切换数据库，若没有该数据库会自动创建，前提是里面要有数据（才会显示）

#### 增

1. `db.test.insert({x:1})`向当前数据库的test表中插入一条数据
2. `db.test.insertMany([{x:1},{x:2},{x:3}])`向当前数据库的test表中插入多条数据

#### 查

1. `db.test.find()`查询当前所有的文档
2. `db.test.findOne()`查询当前的一个文档

可以看到还有一个`_id`字段，这是系统自动为我们添加的字段，我们也可以自己传入`_id`，但是`_id`字段不能重复

#### 改

`db.test.update({x:1},{x:999})`更新涉及两个参数，一个是更新条件，一个是要更新的数据

#### 删

`db.test.remove({x:999})`删除涉及一个参数，即删除条件

#### shell操作

我们也可以将要执行的脚本放在一个js文件中，在使用shell脚本时指定要执行的js文件，如下：

```
mongo ~/myjs.js
```

## MongoDB数据类型

### 数字

shell默认使用64位浮点型数值

```
db.test.insert({x:3.1415926})
db.test.insert({x:3})
```

对于整型值，我们可以使用NumberInt或者NumberLong表示，如下：

```
db.test.insert({x:NumberInt(10)})
db.test.insert({x:NumberLong(12)})
```

### 字符串

字符串也可以直接存储，如下：

```
db.test.insert({x:"hello MongoDB!"})
```

### 正则表达式

正则表达式主要用在查询里边，查询时我们可以使用正则表达式，语法和JavaScript中正则表达式的语法相同，比如查询所有key为x，value以hello开始的文档且不区分大小写`/i`：

```
db.test.find({x:/^(hello)(.[a-zA-Z0-9])+/i})
```

### 数组

数组一样也是被支持的，如下：

```
db.test.insert({x:[1,2,3,4,new Date()]})
```

### 日期

MongoDB支持Date类型的数据，可以直接new一个Date对象，如下：

```
db.test.insert({x:new Date()})
```

### 内嵌文档

一个文档也可以作为另一个文档的value，这个其实很好理解，如下：

```
db.test.insert({name:"三国演义",author:{name:"罗贯中",age:99}})
```

### ObjectId

我们在前面提到过，我们每次插入一条数据系统都会**自动帮我们插入一个`_id`键**，这个键的值不可以重复，它可以是任何类型的，我们也可以手动的插入，默认情况下它的数据类型是ObjectId，由于MongoDB在设计之初就是用作**分布式数据库**，所以使用ObjectId可以避免不同数据库中`_id`的重复（如果**使用自增的方式在分布式系统中就会出现重复的`_id`的值**），这个特点有点类似于Git中的版本号和Svn中版本号的区别。

ObjectId使用12字节的存储空间，每个字节可以存储两个十六进制数字，所以一共可以存储24个十六进制数字组成的字符串，在这24个字符串中，前8位表示时间戳，接下来6位是一个机器码，接下来4位表示进程id，最后6位表示计数器。

### 二进制

MongoDB中也可以存储二进制数据，不过这种情况并不多，二进制数据的存储不能在shell中操作，我们在后面的代码中会介绍这种存储方式。

### 代码

文档中也可以包括JavaScript代码，如下：

```
db.test.insert({x:function f1(a,b){return a+b;}})
```

## 文档更新操作

### 文档替换

#### 集合修改

假设我的集合中现在存了如下一段数据：

```
{
    "_id" : ObjectId("59f005402844ff254a1b68f6"),
    "name" : "三国演义",
    "authorName" : "罗贯中",
    "authorGender" : "男",
    "authorAge" : 99.0
}
```

这是一本书，有书名和作者信息，但是作者是一个独立的实体，所以我想将之提取出来，变成下面这样：

```
{
    "_id" : ObjectId("59f005402844ff254a1b68f6"),
    "name" : "三国演义",
    "author" : {
        "name" : "罗贯中",
        "gender" : "男",
        "age" : 99.0
    }
}
```

操作步骤：

```
//加入文档
db.test.insert({"name" : "三国演义",    "authorName" : "罗贯中",    "authorGender" : "男",    "authorAge" : 99.0})
//获取文档
var book = db.test.findOne({name:"三国演义"})
//修改文档
book.author = {name:book.authorName,gender:book.authorGender,age:book.authorAge}
//删除重复字段
delete book.authorName
delete book.authorGender
delete book.authorAge
//更新文档
db.test.update({name:"三国演义"},book)
```

#### 重复文档更新

另外一个问题是更新时，MongoDB只会匹配第一个更新的文档，假设我的MongoDB中有如下数据：

```
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f7"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f8"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f9"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68fa"), "x" : 2 }
```

我想把所有x为1的数据改为99，我们很容易想到如下命令：

```
db.test.update({x:1},{x:99})
```

但我们发现执行结果却是这样：

```
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f7"), "x" : 99 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f8"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68f9"), "x" : 1 }
{ "_id" : ObjectId("59f00d4a2844ff254a1b68fa"), "x" : 2 }
```

即只有第一条匹配的结果被更新了，其他的都没有变化。这是MongoDB的更新规则，即只更新第一条匹配结果。如果我们想将所有x为1的更新为x为99，可以采用如下命令：

```
db.test.update({x:1},{$set:{x:99}},false,true)
```

首先我们将要修改的数据赋值给set是一个修改器，我们将在下文详细讲解，然后后面多了两个参数，**第一个false表示如果不存在update记录，是否将我们要更新的文档作为一个新文档插入**，true表示插入，false表示不插入，默认为false，**第二个true表示是否更新全部查到的文档**，false表示只更新第一条记录，true表示更新所有查到的文档。

### 使用修改器

很多时候我们修改文档，只是要修改文章的某一部分，而不是全部，但是现在我面临这样一个问题，假设我有如下一个文档：

```
{x:1,y:2,z:3}
```

我现在想把这个文档中x的值改为99，我可能使用如下操作：

```
db.test.update({x:1},{x:99})
```

但是更新结果却变成了这样：

```
{ "_id" : ObjectId("59f02dce95769f660c09955b"), "x" : 99 }
```

#### $set修改器

$set可以用来修改一个字段的值，如果这个字段不存在，则创建它。如下：

```
db.test.update({x:1},{$set:{x:99}})
```

#### $inc修改器

$inc用来增加已有键的值，如果该键不存在就新创建一个。比如我想给上文的罗贯中增加一个年龄为99，方式如下：

```
db.test.update({name:"三国演义"},{$inc:{"author.age":99}})
```

执行结果如下：

```
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 99.0
    }
}
```

加入我想给罗贯中增加1岁，执行如下命令：

```
db.test.update({name:"三国演义"},{$inc:{"author.age":1}})
```

这是会在现有值上加1，结果如下：

```
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    }
}
```

注意$inc只能用来操作数字，不能用来操作null、布尔等。

### 数组修改器

数组修改器有好几种，我们分别来看。
$push可以向已有数组末尾追加元素，要是不存在就创建一个数组，还是以我们的上面的book为例，假设book有一个字段为comments，是一个数组，表示对这个book的评论，我们可以使用如下命令添加一条评论：

```
db.test.update({name:"三国演义"},{$push:{comments:"好书666"}})
```

此时不存在comments字段，系统会自动帮我们创建该字段，结果如下：

```
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "好书666"
    ]
}
```

此时我们可以追加评论，如下：

```
db.test.update({name:"三国演义"},{$push:{comments:"好书666啦啦啦啦"}})
```

结果如下：

```
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "好书666", 
        "好书666啦啦啦啦"
    ]
}
```

如果想一次添加3条评论，可以结合$each一起来使用，如下：

```
db.test.update({name:"三国演义"},{$push:{comments:{$each:["111","222","333"]}}})
```

结果如下：

```
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "好书666", 
        "好书666啦啦啦啦", 
        "111", 
        "222", 
        "333"
    ]
}
```

我们可以使用$slice来固定数组的长度，假设我固定数组的长度为5，如果数组中的元素不足5个，则全部保留，如果数组中的元素超过5个，则只会保留最新的5个，如下：

```
db.test.update({name:"三国演义"},{$push:{comments:{$each:["444","555"],$slice:-5}}})
```

注意$slice的值为负数，运行结果如下：

```
{
    "_id" : ObjectId("59f042cfcafd355da9486008"),
    "name" : "三国演义",
    "author" : {
        "name" : "明代罗贯中",
        "gender" : "男",
        "age" : 100.0
    },
    "comments" : [ 
        "111", 
        "222", 
        "333", 
        "444", 
        "555"
    ]
}
```

我们还可以在清理之前使用$sort对数据先进行排序，然后再清理比如我有一个class文档，如下：

```
{
    "_id" : ObjectId("59f07f3649fc5c9c2412a662"),
    "class" : "三年级二班"
}
```

现在向这个文档中插入student，每个student有姓名和成绩，然后按照成绩降序排列，只要前5条数据，如下：

```
db.test.update({class:"三年级二班"},{$push:{students:{$each:[{name:"张一百",score:100},{name:"张九九",score:99},{name:"张九八",score:98}],$slice:5,$sort:{score:-1}}}})
```

$sort的取值为-1和1，-1表示降序，1表示升序。
上面的命令执行两次之后（即插入两次），结果如下：

```
{
    "_id" : ObjectId("59f07f3649fc5c9c2412a662"),
    "class" : "三年级二班",
    "students" : [ 
        {
            "name" : "张一百",
            "score" : 100.0
        }, 
        {
            "name" : "张一百",
            "score" : 100.0
        }, 
        {
            "name" : "张九九",
            "score" : 99.0
        }, 
        {
            "name" : "张九九",
            "score" : 99.0
        }, 
        {
            "name" : "张九八",
            "score" : 98.0
        }
    ]
}
```

**sort不能只和each。**  

### $addToSet

我们可以在插入的时候使用$addToSet，表示要插入的值如果存在则不插入，否则插入，如下：

```
db.test.update({name:"三国演义"},{$addToSet:{comments:"好书"}})
```

上面的命令执行多次之后，发现只成功插入了一条数据。也可以将each结合起来使用，如下：

```
db.test.update({name:"三国演义"},{$addToSet:{comments:{$each:["111","222","333"]}}})
```

### $pop

$pop可以用来删除数组中的数据，如下：

```
db.test.update({name:"三国演义"},{$pop:{comments:1}})
```

1表示从comments数组的末尾删除一条数据，-1表示从comments数组的开头删除一条数据。

### $pull

使用$pull我们可以按条件删除数组中的某个元素，如下：

```
db.test.update({name:"三国演义"},{$pull:{comments:"444"}})
```

表示删除数组中值为444的数据。

### $

既然是数组，我们当然可以通过下标来访问，如下一行操作表示将下标为0**comments.0**的(第一个comments)comments修改为999：

```
db.test.update({name:"三国演义"},{$set:{"comments.0":"999"}})
```

可是有的时候我并不知道我要修改的数据处于数组中的什么位置，这个时候可以使用$符号来解决：

```
db.test.update({comments:"333"},{$set:{"comments.$":"333-1"}})
```

**comments.$**查询条件**查出来333的下标**，符号就能将之修改。

### save

save是shell中的一个函数，接收一个参数，这个参数就是文档，如果文档中有`_id`参数save会执行更新操作，否则执行插入操作，使用save操作我们可以方便的完成一些更新操作。

类似于如下命令则表示一个插入操作(因为没有`_id`)：

```
db.test.save({x:111})
```

### null

null的查询稍微有点不同，假如我想查询z为null的数据，如下：

```
db.test.find({z:null})
```

这样不仅会查出z为null的文档，也会查出所有没有z字段的文档，如果只想查询z为null的字段，那就再多加一个条件，判断一下z这个字段存在不**$exists:true**，如下：

```
db.test.find({z:{$in:[null],$exists:true}})
```

### 数组查询

假设我有一个数据集如下：

```
{
    "_id" : ObjectId("59f1ad41e26b36b25bc605ae"),
    "books" : [ 
        "三国演义", 
        "红楼梦", 
        "水浒传"
    ]
}
```

查询books中含有三国演义的文档，如下：

```
db.test.find({books:"三国演义"})
```

如果要查询既有三国演义又有红楼梦的文档，可以使用**$all**，如下：

```
db.test.find({books:{$all:["三国演义","红楼梦"]}})
```

当然我们也可以使用精确匹配，比如查询books为`"三国演义","红楼梦", "水浒传"`的数据，如下：

```
db.test.find({books:["三国演义","红楼梦", "水浒传"]})
```

不过这种就会一对一的精确匹配。

也可以按照下标匹配，比如我想查询数组中下标为2的项**books.2**的为`"水浒传"`的文档，如下：

```
db.test.find({"books.2":"水浒传"})
```

也可以按照数组长度来查询，比如我想查询数组长度为3的文档**$size:3**：

```
db.test.find({books:{$size:3}})
```

如果想查询数组中的前两条数据，可以使用**$slice**，如下：

```
db.test.find({},{books:{$slice:2}})
```

注意这里要写在find的第二个参数的位置。**2表示数组中前两个元素**，**-2表示从后往前数两个元素**。也可以截取数组中间的元素，比如查询数组的第二个到第四个元素：

```
db.test.find({},{books:{$slice:[1,3]}})
```

数组中的与的问题也值得说一下，假设我有如下数据：

```
{
    "_id" : ObjectId("59f208bc7b00f982986c669c"),
    "x" : [ 
        5.0, 
        25.0
    ]
}
```

我想将数组中value取值在(10,20)之间的文档获取到，如下操作：

```
db.test.find({x:{$lt:20,$gt:10}})
```

此时上面这个文档虽然不满足条件却依然被查找出来了，因为`5<20`，而`25>10`，要解决这个问题，我们可以使用$elemMatch，如下：

```
db.test.find({x:{$elemMatch:{$lt:20,$gt:10}}})
```

$elemMatch要求MongoDB同时使用查询条件中的两个语句与一个数组元素进行比较。

### 嵌套文档查询

嵌套文档有两种查询方式，比如我的数据如下：

```
{
    "_id" : ObjectId("59f20c9b7b00f982986c669f"),
    "x" : 1.0,
    "y" : {
        "z" : 2.0,
        "k" : 3.0
    }
}
```

想要查询上面这个文档，我的查询语句如下：

```
db.test.find({y:{z:2,k:3}})
```

但是这种写法要求严格匹配，顺序都不能变，假如写成了`db.sang_collect.find({y:{k:3,z:2}})`，就匹配不到了，因此这种方法不够灵活，我们一般推荐的是下面这种写法：

```
db.test.find({"y.z":2,"y.k":3})
```

这种写法可以任意颠倒顺序。

