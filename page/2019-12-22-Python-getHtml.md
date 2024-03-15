---
title: Python图片爬取
date:  	2019-12-22 08:20:36 +0800
category: Original
tags: Python
excerpt: Python网页爬取图片实例
---

### 爬取步骤

1. 定位需要爬取的网页
2. 根据html特征通过正则表达式锁定需要爬取的图片
3. 抓取图片并保存到本地

### 源代码

```
import urllib.request
import re

url = ''	#需要爬取的网页
request = urllib.request.Request(url)
response = urllib.request.urlopen(request)
buf = response.read()
buf = str(buf, encoding='utf-8')
# print(buf)
# 获取所有图片url地址列表
listurl = re.findall(r'src="(.+?\.jpg)"', buf)
print(listurl)

i = 1
for url in listurl:
    f = open(str(i) + '.jpg', 'wb+')	#设置保存的文件名
    req = urllib.request.urlopen(url)
    buf = req.read()
    f.write(buf)
    i += 1
```

### 我的思考

1. 根据所爬取的对象，对图片进行分类保存在不同的文件夹
2. 正则表达式中能够匹配我设定的变量

##### 保存在不同的文件夹

主要涉及到三个函数，首先需要引入`os`包

1. `os.path.exists(path)` 判断一个目录是否存在
2. `os.makedirs(path)` 多层创建目录
3. `os.mkdir(path)` 创建目录

```
def mkdir(path):
    # 去除首位空格
    path = path.strip()
    # 去除尾部 \ 符号
    path = path.rstrip("\\")
    # 判断路径是否存在
    # 存在     True
    # 不存在   False
    isExists = os.path.exists(path)
    # 判断结果
    if not isExists:
        # 如果不存在则创建目录
        # 创建目录操作函数
        os.makedirs(path)
        print
        path + ' 创建成功'
        return True
    else:
        # 如果目录存在则不创建，并提示目录已存在
        print
        path + ' 目录已存在'
        return False
# 定义要创建的目录\
mkpath = "d:\\root\\"
# 调用函数
mkdir(mkpath)
```



##### 匹配变量

查阅资料发现Python可以通过`'+变量+'`的方式将变量引入到正则表达式之中，例如想要匹配图片名称中包含某个ID的图片

```
ID = ''#此处输入需要匹配的ID
listurl = re.findall(r'src="(.+?'+ID+'.+?\.jpg)"', buf)
```

### 应对频繁访问网站的拦截机制

加入sleep函数，在爬取一张网页后对爬虫程序进行休眠，再进行下一张网页的爬取

需要引入time，例如睡眠0.5秒

```
time.sleep(0.5)
```

### 完整代码展示

```
import os
import time
import urllib.request
import re

url = ''
PAGE = 13
#提取ID
ID = re.findall(r'', url)
print(ID)
request = urllib.request.Request(url)
response = urllib.request.urlopen(request)
buf = response.read()
buf = str(buf, encoding='utf-8')


def mkdir(path):
    # 去除首位空格
    path = path.strip()
    # 去除尾部 \ 符号
    path = path.rstrip("\\")
    # 判断路径是否存在
    # 存在     True
    # 不存在   False
    isExists = os.path.exists(path)
    # 判断结果
    if not isExists:
        # 如果不存在则创建目录
        # 创建目录操作函数
        os.makedirs(path)
        print
        path + ' 创建成功'
        return True
    else:
        # 如果目录存在则不创建，并提示目录已存在
        print
        path + ' 目录已存在'
        return False
# 定义要创建的目录\
mkpath = ""
# 调用函数
mkdir(mkpath)

i = 1
for index in range(1,PAGE):
    if(index != 1):
        url = ''
        request = urllib.request.Request(url)
        response = urllib.request.urlopen(request)
        buf = response.read()
        buf = str(buf, encoding='utf-8')
    # print(buf)
    # 获取所有图片url地址列表
    listurl = re.findall(r'src="(.+?' + ID + '.+?\.jpg)"', buf)
    print(listurl)

    for url in listurl:
        f = open(mkpath + str(i) + '.jpg', 'wb+')
        req = urllib.request.urlopen(url)
        buf = req.read()
        # buf = str(buf)
        f.write(buf)
        i += 1
    time.sleep(0.5)
```
