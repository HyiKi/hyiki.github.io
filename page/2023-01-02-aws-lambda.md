---
title: aws之lambda服务学习笔记
date:  2023-01-02 14:59:59 +0800
category: Original
tags: Redis
excerpt: serverless 服务学习
---

0. 配置
   - Linux：
     CentOS Linux 8 (Core)
   - Java：
     java version "11.0.16" 2022-07-19 LTS
   - aws --version：
     aws-cli/2.9.12 Python/3.9.11 Linux/4.18.0-348.7.1.el8_5.x86_64 exe/x86_64.centos.8 prompt/off
   - gradle -v：
     Gradle 7.6

1. 认识 serverless 服务

   - 版本控制：使用版本来**管理函数的部署**。例如您可以发布函数的新版本以用于 Beta 测试，而不会影响稳定生产版本的用户。保证**调用方函数不可变**原则
   - 自动托管：允许在不提供配置或管理服务器的情况下**运行代码**
   - 弹性伸缩：Lambda 只在需要时运行函数，并**根据用户配置与并发量**自动扩展，升级或降级，从每天几个请求到每秒几千个请求
   - 按量计费：按照**请求次数**以及**计算时间**计费，其中免费套餐包括每月 100 万次免费请求和 400000 GB 秒的计算时间

2. 从 Concurrency 角度理解 serverless 的 并发度 / QPS

   参考文档：<https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html>

   1. 假设您有一个函数，平均需要20毫秒才能运行。在高峰负载期间，每秒可以观察到5,000个请求。在峰值负载期间，您的函数的并发度是多少？

      ```sh
      Concurrency = (5,000 requests/second) * (0.020 seconds/request) = 100
      ```

   2. 保留并发度（Reserved concurrency）：为函数保留的并发度，其他函数无法占用

   3. 预置并发度（Provisioned concurrency）：**函数预热**！Lambda 会初始化执行环境，以便它们准备好立即响应函数请求，预置并发里的函数调用都属于热启动

3. 服务场景

   参考文档：<https://docs.aws.amazon.com/lambda/latest/dg/welcome.html?icmpid=docs_lambda_help>

   1. 文件处理：上传文件后，对**文件的处理**
   2. 流处理：**处理实时流数据**，用于应用程序活动跟踪、交易订单处理、点击流分析、数据清理、日志过滤、索引、社交媒体分析、物联网(IoT)设备数据遥测和计量
   3. Web 应用程序： 将 Lambda 与其他 AWS **服务结合**起来，构建功能强大的 Web 应用程序，这些应用程序可以自动升级和降级，并在跨多个数据中心的高可用配置中运行
   4. 物联网后端： 使用 Lambda **构建无服务器后端**来处理 Web、移动、物联网和第三方 API 请求
   5. 移动后端： 使用 Lambda 和 AmazonAPI 网关构建后端，以验证和处理 API 请求。使用 AWS Amplify 可以轻松地将后端集成到 iOS、 Android、 Web 和 React 本机前端

   个人总结就是，**适用于耗时较长，或并发度高，或占用资源不稳定的函数**
   单拎弹性计算这个点，与前端对资源的需求非常契合，根据页面请求数来分配计算资源

4. 服务使用

   仓库：<https://github.com/awsdocs/aws-lambda-developer-guide.git>

   1. 克隆官方Git，拉取lambda开发者向导项目

      ```
      [root@VM-8-11-centos lambda]# git clone https://github.com/awsdocs/aws-lambda-developer-guide.git
      正克隆到 'aws-lambda-developer-guide'...
      remote: Enumerating objects: 10212, done.
      remote: Counting objects: 100% (210/210), done.
      remote: Compressing objects: 100% (153/153), done.
      remote: Total 10212 (delta 60), reused 195 (delta 57), pack-reused 10002
      接收对象中: 100% (10212/10212), 8.46 MiB | 199.00 KiB/s, 完成.
      处理 delta 中: 100% (7863/7863), 完成.
      ```

   2. 安装官方组件 aws-cli

      ```sh
      [root@VM-8-11-centos aws-cli]# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      [root@VM-8-11-centos aws-cli]# unzip awscliv2.zip
      [root@VM-8-11-centos aws-cli]# sudo ./aws/install
      [root@VM-8-11-centos aws-cli]# aws --version
      aws-cli/2.9.12 Python/3.9.11 Linux/4.18.0-348.7.1.el8_5.x86_64 exe/x86_64.centos.8 prompt/off
      ```

   3. aws-cli 配置账号

      地区名（region name）参考链接：<https://docs.aws.amazon.com/general/latest/gr/rande.html>

      ```sh
      [root@VM-8-11-centos blank-java]# aws configure
      AWS Access Key ID [****************ZVH6]:
      AWS Secret Access Key [****************rb8P]:
      Default region name [ap-east-1]: us-east-1
      Default output format [​JSON]:
      [root@VM-8-11-centos blank-java]# aws configure list
            Name                    Value             Type    Location
            ----                    -----             ----    --------
         profile                <not set>             None    None
      access_key     ****************ZVH6 shared-credentials-file
      secret_key     ****************rb8P shared-credentials-file
          region                ap-east-1      config-file    ~/.aws/config
      ```

   4. 执行向导里的脚本

      ```sh
      [root@VM-8-11-centos blank-java]# ./1-create-bucket.sh
      make_bucket: lambda-artifacts-cfbde7fe379b6c74
      [root@VM-8-11-centos blank-java]# ./2-build-layer.sh
      [root@VM-8-11-centos blank-java]# ./3-deploy.sh
      [root@VM-8-11-centos blank-java]# ./4-invoke.sh
      {
          "StatusCode": 200,
          "ExecutedVersion": "$LATEST"
      }
      ```

   5. 定制自己的lambda

      最简单的方式就是把项目拉下来，改写里面的`example.Handler::handleRequest`方法，重新部署服务即可，十分简单~
