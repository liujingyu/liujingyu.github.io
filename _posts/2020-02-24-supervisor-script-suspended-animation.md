---
layout: post
title: supervisor 运行的PHP脚本假死排查
categories: [php]
tags: [supervisor, php, gdb]
fullview: true
markdown: kramdown

---

supervisor status

```
[ ~]$ sudo supervisorctl -c /etc/supervisord.conf status
....bigdata-lodgeunit-state_01   RUNNING   pid 27801, uptime 47 days, 0:50:00
....prepare_09                   RUNNING   pid 5159, uptime 20 days, 11:13:15
```

strace 命令查看一下系统调用情况

```
[]$ strace -p 27801
Process 27801 attached
restart_syscall(<... resuming interrupted call ...>) = 0
rt_sigaction(SIGPIPE, NULL, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, 8) = 0
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN|POLLPRI|POLLRDNORM|POLLRDBAND}], 1, 0) = 0 (Timeout)
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN}], 1, 1000) = 0 (Timeout)
rt_sigaction(SIGPIPE, NULL, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, 8) = 0
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN|POLLPRI|POLLRDNORM|POLLRDBAND}], 1, 0) = 0 (Timeout)
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN}], 1, 1000) = 0 (Timeout)
rt_sigaction(SIGPIPE, NULL, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, 8) = 0
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN|POLLPRI|POLLRDNORM|POLLRDBAND}], 1, 0) = 0 (Timeout)
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN}], 1, 1000) = 0 (Timeout)
rt_sigaction(SIGPIPE, NULL, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, 8) = 0
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN|POLLPRI|POLLRDNORM|POLLRDBAND}], 1, 0) = 0 (Timeout)
rt_sigaction(SIGPIPE, {SIG_IGN, [PIPE], SA_RESTORER|SA_RESTART, 0x7ffff4a56670}, NULL, 8) = 0
poll([{fd=19, events=POLLIN}], 1, 1000^CProcess 27801 detached
 <detached ...>
 )
```

lsof 查看一下文件下运行的服务(socket)

```
[]$ lsof -d 19 |grep 27801
php     27801  www   19u     IPv4          430131931      0t0       TCP localhost:9411->localhost:9411 (ESTABLISHED)
```


gdb查看一下

```

[]$ gdb -p 27801
.....
.....
Loaded symbols for /lib64/libnss_dns.so.2
0x00007ffff4b0cbb0 in __poll_nocancel () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install cyrus-sasl-lib-2.1.26-19.2.el7.x86_64 freetype-2.4.11-11.el7.x86_64 glibc-2.17-105.el7.x86_64 gmp-6.0.0-11.el7.x86_64 keyutils-libs-1.5.8-3.el7.x86_64 krb5-libs-1.13.2-10.el7.x86_64 libcom_err-1.42.9-7.el7.x86_64 libgcc-4.8.5-4.el7.x86_64 libjpeg-turbo-1.2.90-5.el7.x86_64 libpng-1.5.13-5.el7.x86_64 libselinux-2.2.2-6.el7.x86_64 libstdc++-4.8.5-4.el7.x86_64 libxml2-2.9.1-5.el7_1.2.x86_64 nspr-4.10.8-2.el7_1.x86_64 nss-3.19.1-18.el7.x86_64 nss-softokn-freebl-3.16.2.3-13.el7_1.x86_64 nss-util-3.19.1-4.el7_1.x86_64 openldap-2.4.40-8.el7.x86_64 openssl-libs-1.0.1e-42.el7.9.x86_64 pcre-8.32-15.el7.x86_64 xz-libs-5.1.2-12alpha.el7.x86_64 zlib-1.2.7-15.el7.x86_64
(gdb) bt
#0  0x00007ffff4b0cbb0 in __poll_nocancel () from /lib64/libc.so.6
#1  0x00007ffff5db3d49 in Curl_poll () from /usr/local/xzsoft/curl/lib/libcurl.so.4
#2  0x00007ffff5daf0c0 in curl_multi_wait () from /usr/local/xzsoft/curl/lib/libcurl.so.4
#3  0x00007ffff5da942c in curl_easy_perform () from /usr/local/xzsoft/curl/lib/libcurl.so.4
#4  0x0000000000585b49 in zif_curl_exec (execute_data=<optimized out>, return_value=0x7ffff2a14ed0) at /home/xzapps/zhaopeng/php-7.1.15/ext/curl/interface.c:3043
#5  0x000000000083ea80 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1099
#6  0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#7  0x000000000083ed95 in ZEND_DO_FCALL_SPEC_RETVAL_UNUSED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:949
#8  0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#9  0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#10 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#11 0x000000000083ed95 in ZEND_DO_FCALL_SPEC_RETVAL_UNUSED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:949
#12 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#13 0x000000000083ed95 in ZEND_DO_FCALL_SPEC_RETVAL_UNUSED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:949
#14 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#15 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#16 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#17 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#18 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#19 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#20 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#21 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#22 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#23 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#24 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#25 0x000000000083dccf in ZEND_CALL_TRAMPOLINE_SPEC_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1991
#26 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#27 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076
#28 0x00000000007edd9b in execute_ex (ex=<optimized out>) at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:429
#29 0x000000000083e915 in ZEND_DO_FCALL_SPEC_RETVAL_USED_HANDLER () at /home/xzapps/zhaopeng/php-7.1.15/Zend/zend_vm_execute.h:1076

```

代码搜索

```
[]$ grep 'localhost:9411' -r .
./vendor/xiaozhu/tracker/src/Tracker.php:            $host = getenv('ZIP_KIN_HOST') ?: 'localhost:9411';
^C
[]$

```


curl 超时缺少设置

```
$host = getenv('ZIP_KIN_HOST') ?: 'localhost:9411';
$ch   = curl_init("http://".$host."/api/v2/spans");
$body = json_encode($spans);
curl_setopt($ch, CURLOPT_POSTFIELDS, $body);
curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type:application/json']);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$result = curl_exec($ch);
curl_close($ch);
```

新增超时配置项

```
// 在尝试连接时等待的秒数
curl_setopt($curl, CURLOPT_CONNECTTIMEOUT , 120);
// 最大执行时间
curl_setopt($curl, CURLOPT_TIMEOUT, 120);
```
