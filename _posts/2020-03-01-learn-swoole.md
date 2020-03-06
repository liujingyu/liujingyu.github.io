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


再来一个漂亮版本的代码

RPC 编码协议类
```php

<?php

namespace Superman2014\SfSwooleConsole;

/**
 * Class Protocol
 * @package Superman2014\SfSwooleConsole
 *
 *
 * 包结构
 *
 * 字段	字节数	说明
 * 包头	定长	每一个通信消息必须包含的内容
 * 包体	不定长	根据每个通信消息的不同产生变化
 *
 * 其中包头详细内容如下：
 *
 * 字段        字节数 类型  说明
 * pkg_len	    2   ushort	整个包的长度，不超过4K
 * version	    1 	uchar	通讯协议版本号
 * command_id	2 	ushort	消息命令ID
 * result	    2 	short	请求时不起作用；请求返回时使用
 *
 */
class Protocol
{

    const VERSION = '1';

    const HEADER = 7;

    const PACKAGE_LENGTH = 4096;

    public static function partHeader($bodyLen, $version, $commandId, $result)
    {
        return pack("nCns", $bodyLen, $version, $commandId, $result);
    }

    public static function partBody($msg)
    {
        return pack("a". strlen($msg), $msg);
    }

    public static function encode($msg, $commandId, $result = 0)
    {

        return self::partHeader(strlen($msg), self::VERSION, $commandId, $result)
            . self::partBody($msg);

    }

    public static function decode($msg)
    {

        $header = substr($msg, 0, self::HEADER);
        $p = unpack('nbodyLen/Cversion/ncommandId/sresult', $header);
        $len = $p['bodyLen'];
        $bodyPack = unpack("a{$len}body", substr($msg, self::HEADER, $len));
        return array_merge($bodyPack, $p);
    }

    public static function main()
    {
        $msg = self::encode("hello", "1001", 0);

        var_dump(self::decode($msg));

        //
//        array(5) {
//        ["body"]=>
//  string(5) "hello"
//        ["bodyLen"]=>
//  int(5)
//  ["version"]=>
//  int(1)
//  ["commandId"]=>
//  int(1001)
//  ["result"]=>
//  int(0)
//    }
    }
}

//Protocol::main();
```

常量类

```
<?php

namespace Superman2014\SfSwooleConsole;

class TaskConstant
{
    const COMMAND_SET = [
        self::START,
        self::STOP,
        self::RESTART,
        self::RELOAD,
        self::PING,
        self::STATUS,
        self::USAGE,
    ];

    const USER_COMMAND = [
        self::STATUS,
        self::RELOAD,
        self::RESTART,
        self::STOP,
    ];

    const START = 'start';

    const STOP = 'stop';

    const STATUS = 'status';

    const RELOAD = 'reload';

    const RESTART = 'restart';

    const PING = 'ping';

    const USAGE = 'usage';

    const DATA = 'data';

    const COMMAND_ID_LIST = [
        1001 => self::START,
        1002 => self::STOP,
        1003 => self::RESTART,
        1004 => self::RELOAD,
        1005 => self::PING,
        1006 => self::STATUS,
        1007 => self::DATA,
    ];

    public static function commandIdByName($name)
    {
       return array_flip(self::COMMAND_ID_LIST)[$name];
    }
}

```

```
<?php

namespace Superman2014\SfSwooleConsole;

use Swoole\Server;
use Swoole\Process;
use Swoole\Client;
use Swoole\Timer;

class TaskServer
{
    const NAME = 'sf-swoole-console';

    public $config = [
        'worker_num' => 4,
        'task_worker_num' => 4,
        'daemonize' => true,
        'backlog' => 128,
        'log_file' => '/tmp/swoole.log',
        'log_level' => 0,
        'task_ipc_mode' => 3,
        'heartbeat_check_interval' => 5,
        'heartbeat_idle_time' => 10,
        'pid_file' => '/tmp/sf-swoole-console.pid',

        'open_length_check' => true,
        'package_length_type' => 'n',
        'package_length_offset' => 0,
        'package_body_offset' => Protocol::HEADER,
        'package_max_length' => Protocol::PACKAGE_LENGTH,
    ];

    const LISTEN_HOST = '0.0.0.0';

    const MANAGE_HOST = '127.0.0.1';

    const PORT = 9501;

    public function __construct($command)
    {
        switch ($command) {
            case TaskConstant::START:
                $this->start();
                break;
            case TaskConstant::STOP:
                 $this->clientSendCommand()(TaskConstant::STOP);
                break;
            case TaskConstant::STATUS:
                $recv = $this->clientSendCommand()(TaskConstant::STATUS);
                var_dump($recv['body']);
                break;
            case TaskConstant::PING:
                var_dump($this->clientSendCommand()(TaskConstant::PING));
                break;
            case TaskConstant::RELOAD:
                $recv = $this->clientSendCommand()( TaskConstant::RELOAD);
                echo $recv['body'],PHP_EOL;
                break;
            case TaskConstant::RESTART:
                $recv = $this->clientSendCommand()( TaskConstant::RESTART);
                echo $recv['body'],PHP_EOL;
                break;
        }
    }

    public function clientSendCommand()
    {
        return function ($command) {
            $client = new Client(SWOOLE_SOCK_TCP);
            if (!$client->connect(self::MANAGE_HOST, self::PORT, -1)) {
                exit("connect failed. Error: {$client->errCode}\n");
            }

            if (in_array($command, TaskConstant::COMMAND_SET)) {
                $client->send(Protocol::encode($command, TaskConstant::commandIdByName($command)));
            } else {
                $client->send(Protocol::encode($command, TaskConstant::commandIdByName(TaskConstant::DATA)));
            }

            $recv = $client->recv();
            $client->close();
            return Protocol::decode($recv);
        };
    }

    public function start()
    {
        $server = new Server(self::LISTEN_HOST, self::PORT, SWOOLE_BASE, SWOOLE_SOCK_TCP);

        $server->set($this->config);

        $server->on('Connect', [$this, 'onConnect']);
        $server->on('Close', [$this, 'onClose']);
        $server->on('Task', [$this, 'onTask']);
        $server->on('Finish', [$this, 'onFinish']);
//        $server->on('Start', [$this, 'onStart']); // SWOOLE_BASE 不存在
        $server->on('WorkerStart', [$this, 'onWorkerStart']);
        $server->on('WorkerStop', [$this, 'onWorkerStop']);
        $server->on('ManagerStart', [$this, 'onManagerStart']);
        $server->on('Shutdown', [$this, 'onShutdown']);
        $server->on('WorkerExit', [$this, 'onWorkerExit']);

        /**
         * 用户进程实现了广播功能，循环接收unixSocket的消息，并发给服务器的所有连接
         */
        $process = new Process(
            function ($process) use ($server) {
                $socket = $process->exportSocket();
                while (true) {
                    $msg = $socket->recv();

                    $command = TaskConstant::COMMAND_ID_LIST[$msg];
                    if ($command == TaskConstant::STATUS) {
                        $socket->send(json_encode($server->stats()));
                    } elseif ($command == TaskConstant::RELOAD) {
                        $server->reload(true);
                        $socket->send('reload ok');
                    } elseif ($command == TaskConstant::RESTART) {
                        Process::kill($server->manager_pid, SIGUSR1);
                        Process::kill($server->manager_pid, SIGUSR2);
                        $socket->send('restart ok');
                    } elseif ($command == TaskConstant::STOP) {
                        $server->shutdown();
                    }
                }
            },
            false,
            2,
            1
        );

        $server->addProcess($process);

        $server->on('Receive', function ($server, $fd, $reactorId, $data) use ($process) {
            $msg = Protocol::decode($data);
            if (in_array($c = TaskConstant::COMMAND_ID_LIST[$msg['commandId']], TaskConstant::USER_COMMAND)) {
                $socket = $process->exportSocket();
                $socket->send($msg['commandId']);
                $server->send($fd, Protocol::encode($socket->recv(), TaskConstant::commandIdByName(TaskConstant::DATA)));
            } else {
                $this->onReceive($server, $fd, $reactorId, $msg);
            }
        });

        $server->start();
    }

    //此回调函数在worker进程中执行
    public function onReceive(Server $server, int $fd, int $reactorId, $data)
    {
        $server->send($fd, Protocol::encode($data['body'], $data['commandId']));
    }

	// ....
}
```

`Server` 的基本基本命令`stop` `restart` `ping` `stop` 就是使用协议里面定义的commandId。 后续还会这里进行深入研究。

跟编解码类似的如编程中序列化，反序列化. 可以看看这边文档 [序列化和反序列化](https://www.infoq.cn/article/serialization-and-deserialization)。




## 参考

* [Swoole 4.x LTS 速查表](https://toxmc.github.io/swoole-cs.github.io/)
* [服务端异步风格](https://wiki.swoole.com/#/server/init)
* [通过swoole理解TCP粘包](https://opso.coding.me/post/swoole-tcp-pack/)
* [理解字节序](https://www.ruanyifeng.com/blog/2016/11/byte-order.html)
* [PHP: 深入pack/unpack](https://my.oschina.net/goal/blog/195749)
* [序列化和反序列化](https://www.infoq.cn/article/serialization-and-deserialization)
* [自定义swoole协议demo](https://github.com/superman2014/sf-swoole-console/tree/feature-self-protocol)
* [msgpack rpc](https://github.com/msgpack-rpc/msgpack-rpc)
