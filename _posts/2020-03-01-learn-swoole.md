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

### 自定义协议

#### EOF结束符

发送者和接收者约定数据包已一个特殊的结束符(EOF)做结尾。适合协议相对简单的需求，常见的比如 redis、memcache、ftp、stmp等都是用\r\n换行符作为结束符。

### 固定包头+包体协议

![固定包头+包体协议](https://opso.coding.me/images/swoole_pack.png)

```php
<?php
// server.php
$serv = new Swoole\Server("127.0.0.1", 9501);
$serv->set(array(
    'open_length_check' => true,
    'package_length_type' => 'n',
    'package_length_offset' => 6,
    'package_body_offset' => 8,
    'package_max_length' => 2000,
));
$serv->on('Connect', function ($serv, $fd) {
    echo "Client: Connect.\n";
});
$serv->on('Receive', function ($serv, $fd, $from_id, $data) {
	$header = substr($data, 0, 8);
	$p = unpack('a6begin/nbodyLen', $header);
	if ($p['begin'] != 'SWOOLE'){
		return;
	}
	$len = $p['bodyLen'];
	$bodyPack = unpack("a{$len}body", substr($data, 8, $len));
    $serv->send($fd, "Server: ".$bodyPack['body']."\n");
});
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});
$serv->start();
```

```php
<?php
//client.php
$client = new swoole_client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501)) {
	exit("connect failed. Error: {$client->errCode}\n");
}
$msg = 'Hello World!';
$client->send(sendMsg($msg)); // 正常发包
$client->send(sendMsg($msg.'0').sendMsg($msg.'1').sendMsg($msg.'2'));// 模拟粘包
echo $client->recv();
$client->close();
function sendMsg($msg) {
	$p = 'SWOOLE';
	$p .= pack('n', strlen($msg));
	$p .= pack('a' . strlen($msg), $msg);
	return $p;
}
```


抓包数据分析

```
13:36:39.050774 IP 127.0.0.1.33844 > 127.0.0.1.9501: Flags [P.], seq 1:21, ack 1, win 342, options [nop,nop,TS val 4907685 ecr 4907685], length 20
        0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
        0x0010:  0048 a56a 4000 4006 9743 7f00 0001 7f00  .H.j@.@..C......
        0x0020:  0001 8434 251d 496d 47ce 8446 849b 8018  ...4%.ImG..F....
        0x0030:  0156 fe3c 0000 0101 080a 004a e2a5 004a  .V.<.......J...J
        0x0040:  e2a5 5357 4f4f 4c45 000c 4865 6c6c 6f20  ..SWOOLE..Hello.
        0x0050:  576f 726c 6421                           World!
13:36:39.050820 IP 127.0.0.1.9501 > 127.0.0.1.33844: Flags [.], ack 21, win 342, options [nop,nop,TS val 4907685 ecr 4907685], length 0
        0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
        0x0010:  0034 663e 4000 4006 d683 7f00 0001 7f00  .4f>@.@.........
        0x0020:  0001 251d 8434 8446 849b 496d 47e2 8010  ..%..4.F..ImG...
        0x0030:  0156 fe28 0000 0101 080a 004a e2a5 004a  .V.(.......J...J
        0x0040:  e2a5                                     ..
13:36:39.050862 IP 127.0.0.1.33844 > 127.0.0.1.9501: Flags [P.], seq 21:84, ack 1, win 342, options [nop,nop,TS val 4907685 ecr 4907685], length 63
        0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
        0x0010:  0073 a56b 4000 4006 9717 7f00 0001 7f00  .s.k@.@.........
        0x0020:  0001 8434 251d 496d 47e2 8446 849b 8018  ...4%.ImG..F....
        0x0030:  0156 fe67 0000 0101 080a 004a e2a5 004a  .V.g.......J...J
        0x0040:  e2a5 5357 4f4f 4c45 000d 4865 6c6c 6f20  ..SWOOLE..Hello.   //这里
        0x0050:  576f 726c 6421 3053 574f 4f4c 4500 0d48  World!0SWOOLE..H
        0x0060:  656c 6c6f 2057 6f72 6c64 2131 5357 4f4f  ello.World!1SWOO
        0x0070:  4c45 000d 4865 6c6c 6f20 576f 726c 6421  LE..Hello.World!
        0x0080:  32                                       2
13:36:39.050885 IP 127.0.0.1.9501 > 127.0.0.1.33844: Flags [.], ack 84, win 342, options [nop,nop,TS val 4907685 ecr 4907685], length 0
        0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
        0x0010:  0034 663f 4000 4006 d682 7f00 0001 7f00  .4f?@.@.........
        0x0020:  0001 251d 8434 8446 849b 496d 4821 8010  ..%..4.F..ImH!..
        0x0030:  0156 fe28 0000 0101 080a 004a e2a5 004a  .V.(.......J...J
        0x0040:  e2a5                                     ..
13:36:39.051626 IP 127.0.0.1.9501 > 127.0.0.1.33844: Flags [P.], seq 1:22, ack 84, win 342, options [nop,nop,TS val 4907685 ecr 4907685], length 21
        0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
        0x0010:  0049 6640 4000 4006 d66c 7f00 0001 7f00  .If@@.@..l......
        0x0020:  0001 251d 8434 8446 849b 496d 4821 8018  ..%..4.F..ImH!..
        0x0030:  0156 fe3d 0000 0101 080a 004a e2a5 004a  .V.=.......J...J
        0x0040:  e2a5 5365 7276 6572 3a20 4865 6c6c 6f20  ..Server:.Hello. // 这里
        0x0050:  576f 726c 6421 0a                        World!.


```



## 参考

* [Swoole 4.x LTS 速查表](https://toxmc.github.io/swoole-cs.github.io/)
* [服务端异步风格](https://wiki.swoole.com/#/server/init)
* [通过swoole理解TCP粘包](https://opso.coding.me/post/swoole-tcp-pack/)
* [理解字节序](https://www.ruanyifeng.com/blog/2016/11/byte-order.html)
