---
layout: post
title: 学习Swoole
categories: [swoole]
tags: [swoole]
fullview: true
markdown: kramdown

---

### 学习Swoole需要掌握哪些基础知识

#### 多进程/多线程

* 了解Linux操作系统进程和线程的概念
* 了解Linux进程/线程切换调度的基本知识
* 了解进程间通信的基本知识，如管道、UnixSocket、消息队列、共享内存

#### SOCKET

* 了解SOCKET的基本操作如accept/connect、send/recv、close、listen、bind
* 了解SOCKET的接收缓存区、发送缓存区、阻塞/非阻塞、超时等概念


#### IO复用

* 了解select/poll/epoll
* 了解基于select/epoll实现的事件循环，Reactor模型
* 了解可读事件、可写事件


#### TCP/IP网络协议

* 了解TCP/IP协议
* 了解TCP、UDP传输协议

#### 调试工具

* 使用 gdb 调试Linux程序
* 使用 strace 跟踪进程的系统调用
* 使用 tcpdump 跟踪网络通信过程
* 其他Linux系统工具，如ps、lsof、top、vmstat、netstat、sar、ss等

概括一句话读《linux高性能服务器编程》

### Swoole 图

#### 类图

![swoole class](/assets/media/WX20200301-072210@2x.png)

#### 运行流程图

![运行流程图](https://wiki.swoole.com/_images/server/running_process.jpg)

#### 进程 / 线程结构图

![进程 / 线程结构图](https://wiki.swoole.com/_images/server/process_structure.jpg)

![结构图2](https://wiki.swoole.com/_images/server/process_structure_2.png)


### 实践代码

可以用代码中作"基酒" -> [sf-swoole-console](https://github.com/superman2014/sf-swoole-console)

Docker运行环境：[https://hub.docker.com/r/phpswoole/swoole](https://hub.docker.com/r/phpswoole/swoole)

简单写了一个mapreduce 的add 加法:[feature-op-add](https://github.com/superman2014/sf-swoole-console/tree/feature-op-add)

```
docker run -p 9501:9501 -it -v $(pwd):/var/html phpswoole/swoole  /bin/bash
cd /var/html
php console task:server start
```

```
$ telnet 127.0.0.1 9501
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
{"op":"add", "data":[1,2]}
3
```



## 参考

* [Swoole 4.x LTS 速查表](https://toxmc.github.io/swoole-cs.github.io/)
* [服务端异步风格](https://wiki.swoole.com/#/server/init)
