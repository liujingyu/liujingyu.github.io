---
layout: post
title: HTTP
categories: [C Protocol]
tags: [c,protocol]
fullview: true
markdown: kramdown

---

## HTTP是什么

HTTP是Hypertext Transfer Protocol 的缩写，超文本传输协议

## 为什么学习HTTP

网络应用程序，分为前端和后端两个部分。当前的发展趋势，就是前端设备层出不穷（手机、平板、桌面电脑、其他专用设备......）。RESTful API是目前比较成熟的一套互联网应用程序的API设计理论, 底层协议正是使用的Http协议。做为后端开发的同学，有必要了解底层的HTTP协议内容及实现逻辑，有利于我们清楚整个具体过程到底发生了什么。

## 如何学习HTTP

### 经典开源的HTTP Server项目

- [源码Tinyhttpd](https://github.com/EZLippi/Tinyhttpd)

- ![流程图](/assets/media/img-blog.csdn.net/20160413230616951.png)

- 阅读代码顺序 `main -> startup -> accept_request -> execute_cgi`

- 每个函数作用
```
accept_request:  处理从套接字上监听到的一个 HTTP 请求，在这里可以很大一部分地体现服务器处理请求流程。
bad_request: 返回给客户端这是个错误请求，HTTP 状态吗 400 BAD REQUEST.
cat: 读取服务器上某个文件写到 socket 套接字。
cannot_execute: 主要处理发生在执行 cgi 程序时出现的错误。
error_die: 把错误信息写到 perror 并退出。
execute_cgi: 运行 cgi 程序的处理，也是个主要函数。
get_line: 读取套接字的一行，把回车换行等情况都统一为换行符结束。
headers: 把 HTTP 响应的头部写到套接字。
not_found: 主要处理找不到请求的文件时的情况。
sever_file: 调用 cat 把服务器文件返回给浏览器。
startup: 初始化 httpd 服务，包括建立套接字，绑定端口，进行监听等。
unimplemented: 返回给浏览器表明收到的 HTTP 请求所用的 method 不被支持。
```

- Docker C 环境, 调试PHP源码用的Docker环境, 大家可以试试 [docker-debug-php-src](https://github.com/liujingyu/docker-debug-php-src)

- [GDB 工具参考](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html)

- [Make 命令教程](http://www.ruanyifeng.com/blog/2015/02/make.html)

## 参考链接

* [RFC文档](https://datatracker.ietf.org/doc/rfc2616/)
    - 摘要
	-    The Hypertext Transfer Protocol (HTTP) is an application-level
    protocol for distributed, collaborative, hypermedia information
    systems. It is a generic, stateless, protocol which can be used for
    many tasks beyond its use for hypertext, such as name servers and
    distributed object management systems, through extension of its
    request methods, error codes and headers [47]. A feature of HTTP is
    the typing and negotiation of data representation, allowing systems
    to be built independently of the data being transferred.

    HTTP has been in use by the World-Wide Web global information
    initiative since 1990. This specification defines the protocol
    referred to as "HTTP/1.1", and is an update to RFC 2068 [33].

 * [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

 * [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)

 * [Tinyhttpd for Ubuntu 14.04 中文详细注释版](https://blog.csdn.net/u013644957/article/details/51147723)

