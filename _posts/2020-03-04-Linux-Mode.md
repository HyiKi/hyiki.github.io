---
title: Linux下的用户态与内核态
date:  2020-03-04 03:49:33 +0800
category: Reference
tags: [Java,Linux]
excerpt: 深入IO并发编程，为了进行学习IO多路复用select、poll、epoll，和Linux中涉及epoll应用的JavaNIO打下基础，多线程上下文切换涉及的用户态与内核态切换的总结。
---

# 用户态与内核态

指的是程序的运行状态，操作系统为了保护自己，不允许用户程序直接访问外部资源，只能通过内核来进行访问。当用户程序需要访问外部资源的时候就需要从用户态切换到内核态，当资源使用完毕又会退回到用户态。

外部资源比较笼统，其实所有的硬件如ram/ssd/显卡声卡/USB都是外部资源。

# 用户态->内核态

三种情况会导致这个切换：

- 系统调用
- 中断
- 异常

例如读文件是对外部资源磁盘的使用，是通过open/read系统调用进入的内核态；再比如申请内存malloc是对外部资源堆内存的使用，是通过brk/mmap系统调用进入的内核态。

# 系统调用

系统调用绝大多数都是c语言可以直接使用的函数，分为5类

- 进程控制 (exit/fork)
- 文件操作 (chmod/chown/read)
- 设备操作 (read/ioctl)
- 信息维护 (getcpu)
- 通信 (pipe/mmap)

`select/poll/epoll`以及`accept/bind`等都是系统调用。在linux下可以通过`man syscalls`查看当前内核支持的所有系统调用。