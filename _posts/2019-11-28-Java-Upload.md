---
title: Java服务端上传图片
date:  2019-11-28 17:20:17 +0800
category: Original
tags: Java
excerpt: 上传图片+java 后端接收实例
---

### 导入包

```
<dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.2</version>
</dependency>
```

### 后端成功代码

```
package com.example.weixin.controller;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import net.sf.json.JSONObject;
import org.apache.commons.fileupload.disk.DiskFileItem;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

@RestController
@RequestMapping("/upload")
public class uploadController {

    //小程序上传照片到后台并返回路径
    @PostMapping("/img")
    public JSONObject uploadFile(HttpServletResponse response, HttpServletRequest request,@RequestParam MultipartFile file) throws IOException {
        Map map = new HashMap();//temp
        map.put("msg","error occur.");
        map.put("status",-1);
//        String realPath = request.getSession().getServletContext().getRealPath("/images");
        //获取文件需要上传到的路径
        File directory = new File("./webapps/biyeji");
        String realPath = directory.getCanonicalPath() + "/images";

        // 判断存放上传文件的目录是否存在（不存在则创建）
        File dir = new File(realPath);
        if (!dir.exists()) {
            dir.mkdir();
        }

        try {
            if(file == null){
                map.put("msg","图片文件不能为空");
                map.put("status",-2);
            }
            else{
//                CommonsMultipartFile cf = (CommonsMultipartFile) file;
//                DiskFileItem fi = (DiskFileItem) cf.getFileItem();
//                File f1 = fi.getStoreLocation();
                MultipartFile f = file;
                File f1 = null;
                try {
                    File dfile = File.createTempFile("prefix", "_" + f.getOriginalFilename());
                    f.transferTo(dfile);
                    f1 = dfile;
                } catch (IOException e) {
                    e.printStackTrace();
                }
                InputStream ips = new FileInputStream(f1);
                String id = "test";
                String path = realPath + "/" + id+ ".jpg";
                OutputStream ops = new FileOutputStream(path);
                byte[] b = new byte[1024];
                int len;
                try {
                    while ((len = ips.read(b)) != -1) {
                        ops.write(b, 0, len);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    // 完毕，关闭所有链接
                    try {
                        ops.close();
                        ips.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                map.put("path","images/"+id+".jpg");
                map.put("msg","图片上传成功");
                map.put("status",0);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        JSONObject json = JSONObject.fromObject(map);
        return json;
    }
}
```

### 遇到的坑

`MultipartFile`不能强转`CommonsMultipartFile`

目的：

使用MultipartFile接收http post传进来的图片，Controller中把接收到的图像转换为File对象给后续程序使用。

报错：

`java.lang.ClassCastException: org.springframework.web.multipart.support.StandardMultipartHttpServletRequest$StandardMultipartFile cannot be cast to org.springframework.web.multipart.commons.CommonsMultipartFile`

原代码：

```
CommonsMultipartFile cf = (CommonsMultipartFile) file;
DiskFileItem fi = (DiskFileItem) cf.getFileItem();
File f1 = fi.getStoreLocation();
```

修改后的代码：

```
MultipartFile f = file;
File f1 = null;
try {
	File dfile = File.createTempFile("prefix", "_" + f.getOriginalFilename());
	f.transferTo(dfile);
	f1 = dfile;
} catch (IOException e) {
	e.printStackTrace();
}
```

总结解决方案：

先创建一个新的临时文件dfile对象，然后调用MultipartFile.transferTo(File dfile)方法把MultipartFile转换为File对象



`MultipartFile`传值`text`报错400



### application.yml配置事例

```
servlet:
  multipart:
    max-file-size: 100MB
    max-request-size: 100MB
    enabled: true
```

### 服务器照片调取实现

把图片置入tomcat文件夹的`webapps`中相应的项目即可，值得注意的是在linux服务器上跑tomcat获取的路径是`/usr/local/tomcat`，因此相应创建图片目录的路径应该为`./webapps/项目名/文件名`，上传图片之后直接在网址中输入相应的路径就能看到图片啦！



> https://blog.csdn.net/qq__1358521215/article/details/78884334
>
> https://blog.csdn.net/ramesha/article/details/84925390
### 过滤上传图片格式

```
String IMG_TYPE_PNG = "PNG";
String IMG_TYPE_JPG = "JPG";
String IMG_TYPE_JPEG = "JPEG";

String suffix = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf(".") + 1);
if(!(IMG_TYPE_JPEG.equals(suffix.toUpperCase()) || IMG_TYPE_JPG.equals(suffix.toUpperCase()) || IMG_TYPE_PNG.equals(suffix.toUpperCase()))){
    map.put("msg","图片文件格式不匹配");
    map.put("status",-3);
}
```