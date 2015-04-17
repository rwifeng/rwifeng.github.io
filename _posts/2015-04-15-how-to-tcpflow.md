---
layout: post
title:  "tcpflow 的简单应用"
date:   2015-04-16 10:07:27
categories: jekyll update
---

大家知道tcpdump 是一个很方便的抓包工具， 但是tcpdump是按照包为单位进行输出的，阅读起来不是很方便。  而tcpflow是面向tcp流的。 每个tcp传输会保存成一个文件，所以一个典型的tcp回话会产生两个文件，每个方向产生一个文件。 并且tcpflow还可以解析tcpdump保存的文件。

### 安装
具体可以参考： https://github.com/simsong/tcpflow


### 简单使用

具体的使用方法，可以去查应用手册`man tcpflow`. 下面介绍几种简单的使用方法。


* 打印经过网卡的所有报文


在你的terminal中输入:

```
 tcpflow -ci en0
```
其中 -c 表示将报文直接打印在terminal中。不指定 -c 参数会将抓取的tcp报文保存在文件中。
-i 表示你要监听的网络端口。 假如你不指定-c 参数， tcpflow会将每个tcp流的数据存储在他自己的文件中，其中文件的命名规则你可以参考: https://github.com/simsong/tcpflow . 基本格式为：

```
  [timestampT]sourceip.sourceport-destip.destport[--VLAN][cNNNN]
```

* 读取已有的 pcap 文件

在使用tcpdump， 或者 wireshark 抓得包的格式是`pcap`。 使用tcpflow也可以读取这些包得格式。
这儿有一个我使用tcpdump 抓得包得格式： http://rwxf.qiniudn.com/test.pcap

```
tcpflow -cr test.pcap
```
即可以将 pcap 包，打印到terminal中。当然你也可以将tcp流保存在文件中：

 ```
tcpflow -c test.pcap
 
这样我得到了两个文件：
	183.136.139.016.00080-192.168.199.146.49570
	192.168.199.146.49570-183.136.139.016.00080
 ```

其中文件 `192.168.199.146.49570-183.136.139.016.00080` 是我主机向服务器发送的请求包。文件 `183.136.139.016.00080-192.168.199.146.49570`是服务器向客户端返回的响应包。如果你想要重放下请求，就可以使用这种方法。

```
nc -i 1 183.136.139.16 80 < 192.168.199.146.49570-183.136.139.016.00080
```

* 使用表达式过滤抓包

tcpflow 也是支持表达式过滤的，格式和tcpdump一样，具体格式和对应参数你可以参考：http://linux.die.net/man/7/pcap-filter 。 这里列集中常用的表达式：


```

过滤经过 192.168.1.202 的流量：
tcpflow -i any host 10.0.3.1

过滤从主机 192.168.1.202 发出的流量：
tcpflow -i any src host 10.0.3.1

过滤从主机 192.168.1.202 发出的流量并且端口号为 80：
tcpflow -i any src host 10.0.3.1 and port 80

过滤固定端口的流量：
tcpflow -i en0 port 443 or port 80

过滤主机 192.168.1.202 端口为80或443的流量：
tcpflow -i en0 'host 192.168.1.202 and (port 443 or port 80)'

...

```