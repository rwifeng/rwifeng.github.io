---
layout: post
title:  "docker haproxy"
date:   2017-04-08
categories: jekyll update
---
本次介绍下怎么通过设置一个haproxy代理来上传到七牛云

Dockerfile:

    FROM haproxy:1.7
    COPY haproxy-up.cfg /usr/local/etc/haproxy/haproxy.cfg


我们看下haproxy的配置文件 haproxy-up.cfg

    global
        daemon
        maxconn 256

    defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms

    frontend http-in
        bind *:80
        http-response add-header X-Up-Server %b/%s
        default_backend servers

    backend servers
        http-request set-header Host up.qiniu.com
        timeout http-keep-alive 3000
        server svr-up up.qiniu.com:80


然后我们构建一个docker镜像：

    $docker build . -t haproxy-up

然后将该镜像推送到七牛：

    $kirk images push haproxy-up

接下来我们我们在七牛将代理服务启动起来：

    $kirk services run -i "rwifeng/haproxy-up:latest" haproxy-up


然后创建一个公网ip映射到我们的代理服务：

    $kirk maps set-port 61.153.154.165:80 --proto tcp -b "default/haproxy-up" --backend-port 80



现在一个完整的代理服务跑起来了， 接下来我们测试下，通过这个服务上传一个字符串到七牛：

    curl -v 61.153.154.165 -F"token=xozWSPMxkMjIVoHg2JyXq4-7-oJaEADLOKHVR0vU:nQcJ-8hxWl7gRB0W8b4GXEomi1M=:eyJzY29wZSI6Impzc2RrIiwiZGVhZGxpbmUiOjE0OTE2NTA5NDl9" -F'file=abc'
    * Rebuilt URL to: 61.153.154.165/
    *   Trying 61.153.154.165...
    * TCP_NODELAY set
    * Connected to 61.153.154.165 (61.153.154.165) port 80 (#0)
    > POST / HTTP/1.1
    > Host: 61.153.154.165
    > User-Agent: curl/7.52.1
    > Accept: */*
    > Content-Length: 358
    > Expect: 100-continue
    > Content-Type: multipart/form-data; boundary=------------------------1a7cb3f6d08dc5ee
    >
    < HTTP/1.1 100 Continue
    < HTTP/1.1 200 OK
    < Access-Control-Allow-Headers: X-File-Name, X-File-Type, X-File-Size
    < Access-Control-Allow-Methods: OPTIONS, HEAD, POST
    < Access-Control-Allow-Origin: *
    < Access-Control-Expose-Headers: X-Log, X-Reqid
    < Access-Control-Max-Age: 2592000
    < Cache-Control: no-store, no-cache, must-revalidate
    < Content-Length: 76
    < Content-Type: application/json
    < Date: Sat, 08 Apr 2017 10:40:00 GMT
    < Pragma: no-cache
    < Server: nginx/1.10.2
    < X-Content-Type-Options: nosniff
    < X-From-Pod: 192.168.160.96
    < X-Log: body;s.ph;s.put.tw;s.put.tr;s.put.tw;s.put.tr;s.ph;s.put.tw;s.put.tr;s.ph;PFDS;PFDS;PFDS:1;rs31_1.sel/not found;rdb.g;bs.r.39.224.43702764868;DBD;v4.get:1;rwro.ins:2/same entry;rs31_1.sel/not found;rdb.g:1;bs.r.39.224.43702764868;DBD:1;v4.get:1;rwro.get:2;MQ;RS.not:;RS:5;rs.put:12;rs-upload.putFile:14;UP:23;UP:24;audit-10.131.214.133:40
    < X-Reqid: GgAAAE107P7zZbMU
    < X-Reqid: GgAAAE107P7zZbMU
    < X-Reqid: GgAAAE107P7zZbMU
    < X-Up-Server: servers/svr-up
    <
    * Curl_http_done: called premature == 0
    * Connection #0 to host 61.153.154.165 left intact
    {"hash":"FqmZPjZHBoFquj4lcXhQwmyc0Nid","key":"FqmZPjZHBoFquj4lcXhQwmyc0Nid"}%


测试成功， 当然这个只是一个简单的demo， 你可以基于这个做很多事情。 或者自己制作其他更有意思的服务。
