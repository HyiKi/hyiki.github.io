---
title: IO多路复用
date:  2020-03-04 04:49:33 +0800
category: Original
tags: [Java,Linux]
excerpt: IO多路复用的学习，以及个人总结
---

* content
{:toc}

### IO多路复用（个人总结）

> IO multiplexing这个词可能有点陌生，但是如果我说select/epoll，大概就都能明白了。有些地方也称这种IO方式为**事件驱动IO**(event driven IO)。我们都知道，select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

![Linux-select-epoll](..\assets\img\Linux-select-epoll.png)

用户态：用户空间，用户可以操作访问

内核态：内核空间，存放着系统内核接口等

在多个客户端对应一台服务器时，更适合IO多路复用的单线程模式处理，因为多个客户端在多线程模式中会频繁地发生线程的上下文切换，而线程调度是在内核态运行，程序代码是在用户态中执行，那么用户态和内核态的相互切换的频繁会很大程度上增大系统的开销。

`在Linux当中每一个网络连接都是以文件描述符fd的形式表示`

#### select

* 调用过程

1. fds数组记录文件描述符，并使用max记录最大的文件描述符值
2. 将文件描述符值在bitmap中对应的0置为1，告知select函数监听max+1个bitmap
3. 调用select函数`select(文件描述符数组,读文件描述符,写文件描述符,异常文件描述符,超时时间)`，此时程序进入阻塞状态，处于程序阶段的用户态会拷贝一份bitmap置于内核态监听，当有一个或多个数据返回时bitmap会发生置位，此时内核态会给用户态反馈
4. 用户态拿到置位后的bitmap进行对文件描述符数组的遍历，从而读/写操作，并处理

* 优点

1. 内核态监听bitmap数组效率高

* 缺点

1. bitmap的长度最大只有1024，意味着最多的连接监听数只有1024个
2. bitmap不可重用，每次调用select处理后，都需要重新重新设置rset
3. 用户态和内核态的交换会增加系统开销
4. 每次循环遍历的时间复杂度是O（n）

#### poll

* 调用过程

1. pollfds结构体储存`int fd`文件描述符，`short events`期望的事件（POLLIN,POLLOUT），`short revents`连接反馈的作用（实际返回的事件）
2. 调用poll函数`poll(文件描述符结构体数组,数组大小,超时时间)`，此时程序会进入阻塞状态，处于程序阶段的用户态会拷贝一份文件描述符结构体数组置于内核态监听，当有一个或多个数据返回时，内核态中相应的文件描述符结构体中的revents会被设置为对应着实际返回的事件，从而给用户态反馈。
3. 用户态拿到修改后的文件描述符结构体，对原文件描述符结构体数组遍历，通过revents项的与&操作来进行判断读/写操作，并处理

* 优点

1. 文件描述符结构体数组可以重用
2. 最大的连接数量没有限制
3. 内核态监听文件描述符结构体数据效率高

* 缺点

1. 用户态和内核态的切换会增加系统开销
2. 每次循环遍历的时间复杂度为O（n）

#### epoll（JavaNIO）

* 调用过程

1. epoll_event结构体储存`int fd`文件描述符和`short events`期望的事件（EPOLLIN,EPOLLOUT）

2. ```
   int epfd = epoll_create(10);
   epoll_ctl(epfd,EPOLL_CTL_ADD,ev.fd,&ev);
   ```

3. 调用epoll函数`epoll(文件描述符结构体数组,数组大小,超时时间)`，此时程序会进入阻塞状态，处于程序阶段的用户态的文件描述符结构体数组和内核态共用，同时内核态进行监听，当有一个或多个数据返回时，内核态中的文件描述符结构体数组会发生重排序，将有数据返回的数组放在前面，并返回发生数据返回的结构体个数。

4. 用户态根据返回的个数对重排后的结构体数组进行访问，通过events项来进行读/写操作，并处理。

* 优点

1. 文件描述符结构体数组可以重用
2. 最大的连接数量没有限制
3. 内核态监听文件描述符结构体数据效率高
4. 每次循环遍历的时间复杂度是O（1）
