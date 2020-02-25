---
layout: post
title: 学习TCP/IP 协议
categories: [tcpdump]
tags: [linux, tcpdump, tcp, ip]
fullview: true
markdown: kramdown

---


## `TCP/IP` 协议（栈） 是什么

TCP/IP（Transmission Control Protocol/Internet Protocol，传输控制协议/网际协议）是指能够在多个不同网络间实现信息传输的协议簇。TCP/IP协议不仅仅指的是TCP 和IP两个协议，而是指一个由FTP、SMTP、TCP、UDP、IP等协议构成的协议簇， 只是因为在TCP/IP协议中TCP协议和IP协议最具代表性，所以被称为TCP/IP协议。

### 计算机网络体系结构分层
TCP/IP协议在一定程度上参考了OSI的体系结构。OSI模型共有七层，从下到上分别是物理层、数据链路层、网络层、运输层、会话层、表示层和应用层。但是这显然是有些复杂的，所以在TCP/IP协议中，它们被简化为了四个层次。

![计算机网络体系结构分层](/assets/media/690219fae5b0587fa26e2dee545e6200)

OSI 七层模型目前只是一个模型，目前实际网络应用的是TCP/IP 四层模型。

### 缺陷

像OSI模型一样，TCP/IP模型和协议也有自己的问题。
* （1）该模型没有明显地区分服务、接口和协议的概念。因此，对于使用新技术来设计新网络，TCP/IP模型不是一个太好的模板。
* （2）TCP/IP模型完全不是通用的，并且不适合描述除TCP/IP模型之外的任何协议栈。
* （3）链路层并不是通常意义上的一层。它是一个接口，处于网络层和数据链路层之间。接口和层间的区别是很重要的。
* （4）TCP/IP模型不区分物理层和数据链路层。这两层完全不同，物理层必须处理铜缆、光纤和无线通信的传输特征；而数据链路层的工作是确定帧的开始和结束，并且按照所需的可靠程度把帧从一端发送到另一端。

### 两台计算机通过TCP/IP协议通讯的过程如下所示

![tcpip](/assets/media/tcpip.transferlan.png)

### TCP/IP数据包的封装

不同的协议层对数据包有不同的称谓，在传输层叫做段（segment），在网络层叫做数据报（datagram），在链路层叫做帧（frame）。数据封装成帧后发到传输介质上，到达目的主机后每层协议再剥掉相应的首部，最后将应用层数据交给应用程序处理。

![tcpip.datagram.png](/assets/media/tcpip.datagram.png)

#### 以太网 格式

![tcpip.ethernetformat.png](/assets/media/tcpip.ethernetformat.png)

#### ARP格式

![2016-04-12_570cbf887a6d7.png](/assets/media/2016-04-12_570cbf887a6d7.png)

#### IP 格式

![ipformat](/assets/media/tcpip.ipformat.png)

#### TCP 格式

![tcpformat](/assets/media/tcpip.tcpformat.png)

![other choose item](/assets/media/3611655386-5b097d85e838f_articlex.jpeg)

### UDP 格式

![udpformat](/assets/media/tcpip.udpformat.png)

### TCP状态流转图
![TCP](/assets/media/79702-b5828684905e8dc1.webp)

各种状态表示的意思

* CLOSED：表示初始状态
* LISTEN：表示服务器端的某个 socket 处于监听状态，可以接受连接
* SYN_SENT：在服务端监听后，客户端 socket 执行 CONNECT 连接时，客户端发送 SYN 报文，此时客户端就进入 SYN_SENT 状态，等待服务端确认。
* SYN_RCVD：表示服务端接收到了 SYN 报文。
* ESTABLISHED：表示连接已经建立了。
* FIN_WAIT_1：其中一方请求终止连接，等待对方的 FIN 报文。
* FIN_WAIT_2：在 FIN_WAIT_2 之后， 当对方回应 ACK 报文之后，进入该状态。
* TIME_WAIT：表示收到了对方的 FIN 报文，并发送出了 ACK 报文，就等 2MSL 之后即可回到 CLOSED 状态。
* CLOSING：一种罕见状态，发生在发送 FIN 报文之后，本应是先收到 ACK 报文，却先收到对方的 FIN 报文，那么就从 FIN_WAIT_1 的状态进入 CLOSING 状态。
* CLOSE_WAIT：表示等待关闭，在 ESTABLISHED 过渡到 LAST_ACK 的一个过渡阶段，该阶段需要考虑是否还有数据发送给对方，如果没有，就可以关闭连接，发送 FIN 报文，然后进入 LAST_ACK 状态。
* LAST_ACK：被动关闭一方发送 FIN 报文之后，最后等待对方的 ACK 报文所处的状态。
* CLOSED：当收到 ACK 保温后，就可以进入 CLOSED 状态了。



### 跨路由器通讯过程

![transferovernet](assets/media/tcpip.transferovernet.png)

### Multiplexing过程

注意，虽然IP、ARP和RARP数据报都需要以太网驱动程序来封装成帧，但是从功能上划分，ARP和RARP属于链路层，IP属于网络层。虽然ICMP、IGMP、TCP、UDP的数据都需要IP协议来封装成数据报，但是从功能上划分，ICMP、IGMP与IP同属于网络层，TCP和UDP属于传输层。

![multiplexing](assets/media/tcpip.multiplex.png)

## 为什么学习 `TCP/IP` 协议

很多公司招聘JD上，写着网络编程经验，说明实际工作中还是有很大的应用市场(应用:微信、QQ, 领域:IM、车联网、服务治理)。学会了它，可以非常轻松了解，浏览器输入URL Enter背后发生了什么。有了这个基础可以更好的学习Socket编程等等等。

![腾讯JD](/assets/media/WX20200225-180648@2x.png)

## 如何学习 `TCP/IP`

我们利用`tcpdump`工具抓包来分析实际网络请求数据。

本地安装`Redis`

step 0: 查看本地的网卡list

```
$ tcpdump -D
1.en0 [Up, Running]
2.p2p0 [Up, Running]
3.awdl0 [Up, Running]
4.llw0 [Up, Running]
5.utun0 [Up, Running]
6.utun1 [Up, Running]
7.utun2 [Up, Running]
8.utun3 [Up, Running]
9.utun4 [Up, Running]
10.lo0 [Up, Running, Loopback]
11.bridge0 [Up, Running]
12.en1 [Up, Running]
13.en2 [Up, Running]
14.gif0 [none]
15.stf0 [none]
```

step 1: 开启抓包工具，命令

```
tcpdump -w /tmp/logs -i lo0 port 6379 -s0

```

-i lo0 抓取回路网口

port 监听redis的端口

-s0 防止包截断

step 2: 开启`Redis-cli` 发送`ping`命令

```
$ redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>

```


step 3: 停止抓包，用Vim打开

```
tcpdump -r /tmp/logs -n -nn -A -x| vim -

```

-x 以16进制形式展示，便于后面分析


首先分析一下我们的ping命令

报文数据中的 [、]、{ 和 } 是为了方便区分数据，我自己加上的。[]包围的部分为本报文中的 IP 头，{}包围的部分为本报文中的 TCP 头。

```
    09:18:06.471045 IP 127.0.0.1.50510 > 127.0.0.1.6379: Flags [P.], seq 18:32, ack 11469, win 6200, options [nop,nop,TS val 192620081 ecr 192618164], length 14: RESP "ping" //请求的数据
        0x0000:  [4500 0042 0000 4000 4006 0000 7f00 0001
        0x0010:  7f00 0001]{c54e 18eb 7c1d d1ad 9576 97dc
        0x0020:  8018 1838 fe36 0000 0101 080a 0b7b 2631
        0x0030:  0b7b 1eb4}2a31 0d0a 2434 0d0a 7069 6e67
        0x0040:  0d0a

IP 层
a) 0x4 4bit， ip 协议版本 0x4 表示 IPv4。
b) 0x5 4bit，ip首部长度
c) 0x00 8bit，服务类型 TOS 现在大多数的TCP/IP实现都不支持TOS特性 。可以看到，本报文 TOS 字段为全 0
d) 0x0042 16bit， IP 报文总长度 单位字节，换算下来，该数据报的长度为 66 字节，数一下上面的报文，恰好 66B。
从占位数来算， IP 数据报最长为 2^16=65535B，但大部分网络的链路层 MTU（最大传输单元）没有这么大，一些上层协议或主机也不会接受这么大的，故超长 IP 数据报在传输时会被分片
e) 0x0000 16bit，标识  唯一的标识主机发送的每一个数据报。通常每发送一个报文，它的值+1。当 IP 报文分片时，该标识字段值被复制到所有数据分片的标识字段中，使得这些分片在达到最终目的地时可以依照标识字段的内容重新组成原先的数据。
f) 0x4000 3bit 标志 + 13bit 片偏移 3bit 标志对应 R、DF、MF。目前只有后两位有效，DF位：为1表示不分片，为0表示分片。MF：为1表示“更多的片”，为0表示这是最后一片。
13bit 片位移：本分片在原先数据报文中相对首位的偏移位。（需要再乘以8）
g) 0x40 8bit 生存时间TTL IP 报文所允许通过的路由器的最大数量。每经过一个路由器，TTL减1，当为 0 时，路由器将该数据报丢弃。TTL 字段是由发送端初始设置一个 8 bit字段.推荐的初始值由分配数字 RFC 指定。发送 ICMP 回显应答时经常把 TTL 设为最大值 255。TTL可以防止数据报陷入路由循环。
h) 0x06 8bit 协议 指出 IP 报文携带的数据使用的是哪种协议，以便目的主机的IP层能知道要将数据报上交到哪个进程。TCP 的协议号为6，UDP 的协议号为17。ICMP 的协议号为1，IGMP 的协议号为2。该 IP 报文携带的数据使用 TCP 协议，得到了验证。
i) 0x0000 16bit IP 首部校验和
j) 0x7f000001 32bit 源地址
k) 0x7f000001 32bit 目的地址

TCP 层
a) 0xc54e  16bit，源端口
b) 0x11eb 16bit，目的端口
c) 0x7c1dd1ad 32bit，序号
d) 0x957697dc 32bit，确认号
e) 0x8 4bit，TCP 报文首部长度  也叫 offset，其实也就是数据从哪里开始。8 * 4 = 32B,因此该 TCP 报文的可选部分长度为 32 - 20 = 12B，这个资源还是很紧张的！ 同 IP 头部类似，最大长度为 60B。
f) 0b000000 6bit, 保留位 保留为今后使用，但目前应置为 0。
g) 0b011000 6bit，TCP 标志位 上图可以看到，从左到右依次是紧急 URG、确认 ACK、推送 PSH、复位 RST、同步 SYN 、终止 FIN。 从抓包可以看出，该报文是带了 ack 的，所以 ACK 标志位置为 1。
h) 0x1838 16bit，滑动窗口大小 解析得到十进制 6200，跟 tcpdump 解析的 win 字段一致。
i) 0xfe36 16bit，校验和 由发送端填充，接收端对 TCP 报文段执行 CRC 算法，以检验 TCP 报文段在传输过程中是否损坏，如果损坏这丢弃。 检验范围包括首部和数据两部分，这也是 TCP 可靠传输的一个重要保障
j) 0x0000 16bit，紧急指针 仅在 URG = 1 时才有意义，它指出本报文段中的紧急数据的字节数。 当 URG = 1 时，发送方 TCP 就把紧急数据插入到本报文段数据的最前面，而在紧急数据后面的数据仍是普通数据。

TCP可选项
a) 0x01 NOP 填充，没有 Length 和 Value 字段， 用于将TCP Header的长度补齐至 32bit 的倍数
b) 0x01 同上。
c) 0x080a 可选项类型为时间戳，len为 10B，value 为0x0b7b 0x2631 0x0b7b 0x1eb4，加上 0x080a，恰好 10B!
启用 Timestamp Option后，该字段包含2 个 32bit 的Timestamp（TS val 和 TS ecr）。
d) 0x0b7b 0x2631 解析后得到 192620081，恰好与 tcpdump 解析到的 TS 字段的 val一致！
e) 0x0b7b 0x1eb4 解析后得到 192618164，恰好与 tcpdump 解析到的 TS 字段的 ecr一致！

该 IP 报文长度为 66B，IP 头长度为 20B，TCP 头部长度为 32B，因此得到数据的长度为 66 - 20 - 32 = 14B，这与 tcpdump 解析到的 length 字段一致

ascii 表 (man 7 ascii)

0x2a31         -> *1
0x0d0a         -> \r\n
0x2434         -> $4
0x0d0a         -> \r\n
0x7069 0x6e67  -> ping
0x0d0a         -> \r\n

    09:18:06.471069 IP 127.0.0.1.6379 > 127.0.0.1.50510: Flags [.], ack 32, win 6379, options [nop,nop,TS val 192620081 ecr 192620081], length 0
        0x0000:  4500 0034 0000 4000 4006 0000 7f00 0001
        0x0010:  7f00 0001 18eb c54e 9576 97dc 7c1d d1bb
        0x0020:  8010 18eb fe28 0000 0101 080a 0b7b 2631
        0x0030:  0b7b 2631
    09:18:06.471115 IP 127.0.0.1.6379 > 127.0.0.1.50510: Flags [P.], seq 11469:11476, ack 32, win 6379, options [nop,nop,TS val 192620081 ecr 192620081], length 7: RESP "PONG" // 返回的数据
        0x0000:  4500 003b 0000 4000 4006 0000 7f00 0001
        0x0010:  7f00 0001 18eb c54e 9576 97dc 7c1d d1bb
        0x0020:  8018 18eb fe2f 0000 0101 080a 0b7b 2631
        0x0030:  0b7b 2631 2b50 4f4e 470d 0a
    09:18:06.471132 IP 127.0.0.1.50510 > 127.0.0.1.6379: Flags [.], ack 11476, win 6200, options [nop,nop,TS val 192620081 ecr 192620081], length 0
        0x0000:  4500 0034 0000 4000 4006 0000 7f00 0001
        0x0010:  7f00 0001 c54e 18eb 7c1d d1bb 9576 97e3
        0x0020:  8010 1838 fe28 0000 0101 080a 0b7b 2631
        0x0030:  0b7b 2631
```

step4: tcpdump 补充 filter可以简单地分为三类：type, dir 和 proto。

* type 区分报文的类型，主要由 host（主机）, net（网络，支持 CIDR） 和 port(支持范围，如 portrange 21-23) 组成。
* dir 区分方向，主要由 src 和 dst 组成。
* proto 区分协议支持 tcp、udp 、icmp 等。

下面说几个 filter 表达式。

proto[x:y] start at offset x into the proto header and read y bytes

[x] abbreviation for [x:1]

注意：单位是字节，不是位！

打印特定 TCP Flag 的数据包

```
TCP Flags 在 tcpdump 抓取的报文中的体现：
[S]：SYN（开始连接）
[.]: 没有 Flag
[P]: PSH（推送数据）
[F]: FIN （结束连接）
[R]: RST（重置连接）
[S.] SYN-ACK，就是 SYN 报文的应答报文。

tcpdump 'tcp[13] & 16!=0'
# 等价于
tcpdump 'tcp[tcpflags] == tcp-ack'
```



## 实际应用场景



### 参考

* [TCP/IP协议](https://baike.baidu.com/item/TCP%2FIP%E5%8D%8F%E8%AE%AE/212915?fromtitle=tcp%2Fip&fromid=214077)
* [一篇文章带你熟悉 TCP/IP 协议（网络协议篇二）](https://juejin.im/post/5a069b6d51882509e5432656)
* [Linux C编程一站式学习](https://item.jd.com/65489151254.html)
* [rfc894](https://datatracker.ietf.org/doc/rfc894/)
* [TCP/IP协议详解卷一](http://www.kancloud.cn:8080/lifei6671/tcp-ip)
* [从tcpdump抓包看TCP/IP协议](https://segmentfault.com/a/1190000015044878)