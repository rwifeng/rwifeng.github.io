---
layout: post
title:  "dns 介绍"
date:   2015-06-01
categories: jekyll update
---
DNS(Domain Name System)是一个基于TCP/IP应用程序的分布式数据库， 他提供主机名字到ip地址之间的转换。



### 结构

#### DNS的层次组织：
* QFNA  -- Full Qualified Domain Name
* xxx.in-addr.arpa

### 协议

#### DNS 查询和响应的一般格式:


* dig 查询:
	
		dig www.qiniu.com

		; <<>> DiG 9.8.3-P1 <<>> www.qiniu.com
		;; global options: +cmd
		;; Got answer:
		;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56934
		;; flags: qr rd ra; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 0

		;; QUESTION SECTION:
		;www.qiniu.com.			IN	A

		;; ANSWER SECTION:
		www.qiniu.com.		515	IN	CNAME	www.qiniu.com.wscdns.com.
		www.qiniu.com.wscdns.com. 515	IN	CNAME	qiniu.xdwscache.glb0.lxdns.com.
		qiniu.xdwscache.glb0.lxdns.com.	30 IN	A	218.92.227.121
		qiniu.xdwscache.glb0.lxdns.com.	30 IN	A	222.186.132.79
		qiniu.xdwscache.glb0.lxdns.com.	30 IN	A	58.222.19.61
		qiniu.xdwscache.glb0.lxdns.com.	30 IN	A	180.97.211.38
		qiniu.xdwscache.glb0.lxdns.com.	30 IN	A	58.221.78.105

		;; Query time: 13 msec
		;; SERVER: 114.114.114.114#53(114.114.114.114)
		;; WHEN: Tue Jun  2 17:08:28 2015
		;; MSG SIZE  rcvd: 187


* tcpdump `sudo tcpdump -i all host 114.114.114.114`包：


		17:08:28.438644 IP 192.168.200.178.61149 > public1.114dns.com.domain: 56934+ A? www.qiniu.com. (31)
		17:08:28.449578 IP public1.114dns.com.domain > 192.168.200.178.61149: 56934 7/0/0 CNAME www.qiniu.com.wscdns.com., CNAME qiniu.xdwscache.glb0.lxdns.com., A 218.92.227.121, A 222.186.132.79, A 58.222.19.61, A 180.97.211.38, A 58.221.78.105 (187)
		
		
