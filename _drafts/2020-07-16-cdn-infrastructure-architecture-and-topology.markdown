---
layout: post
title: "CDN Infrastructure Architecture and Topology"
---

## Why CDN ?

我们要知道网站的访问速度除了与服务器的硬件性能和带宽有关以外，用户与服务的链路速度也很重要。

一般来说服务器距离用户距离越近访问速度就会越快，当然这个 “近” 可能是地理上的距离，也可能是逻辑上的距离。

实际上，这个距离就是网络中数据包经过的 NAT/SNAT 路由,  一个数据包经过的路由越多被中转的次数越多延时越高，次数越少延时越低。

而造成地理距离和社区规模则是路由投入大的根本原因。

我们可以通过 `traceroute` 命令来查看我们访问一个服务的路由情况：

```
✘130 ➜ traceroute lsong.org
traceroute to lsong.org (185.199.111.153), 64 hops max, 52 byte packets
 1  30.30.199.254 (30.30.199.254)  18.796 ms  28.741 ms  3.916 ms
 2  30.30.253.201 (30.30.253.201)  11.627 ms  11.104 ms  5.022 ms
 3  30.30.253.1 (30.30.253.1)  4.674 ms  8.778 ms  7.827 ms
 4  30.30.254.5 (30.30.254.5)  4.612 ms  4.414 ms  3.127 ms
 5  42.120.240.221 (42.120.240.221)  16.319 ms  21.778 ms  35.463 ms
 6  106.38.196.45 (106.38.196.45)  22.558 ms  12.583 ms
    106.38.196.249 (106.38.196.249)  12.140 ms
 7  220.181.177.73 (220.181.177.73)  14.717 ms
    36.110.244.17 (36.110.244.17)  22.517 ms
    220.181.177.241 (220.181.177.241)  12.737 ms
 8  202.97.34.74 (202.97.34.74)  17.657 ms
    202.97.53.26 (202.97.53.26)  20.352 ms
    202.97.34.158 (202.97.34.158)  7.069 ms
 9  202.97.12.50 (202.97.12.50)  25.988 ms
    202.97.81.138 (202.97.81.138)  26.062 ms *
10  xe-1-0-4.r27.tokyjp05.jp.bb.gin.ntt.net (129.250.66.53)  141.264 ms  131.424 ms  143.569 ms
11  ae-2.r31.tokyjp05.jp.bb.gin.ntt.net (129.250.2.155)  67.279 ms
    ae-1.r30.tokyjp05.jp.bb.gin.ntt.net (129.250.2.157)  75.673 ms
    ae-2.r31.tokyjp05.jp.bb.gin.ntt.net (129.250.2.155)  72.707 ms
12  ae-2.r01.tokyjp08.jp.bb.gin.ntt.net (129.250.6.131)  125.896 ms * *
13  * * *
14  * * *
```

那么由此可见，想要提高网络访问速度只需要在距离用户比较的网络节点设置站点就好了，

举个例子，比如你所在的公司需要经常访问另一个公司的在线文档服务，由于这家公司在国外，所以同事们经常反馈说很慢，有时候甚至打不开

经过沟通，这家公司决定就在你所在的公司内部架设一个服务器节点，然后你会发现速度飞快，再也没有同事抱怨了。

![](https://segmentfault.com/img/remote/1460000019036403)

## 

通过上面的例子，我们大概了解了 CDN 的基本用途，但是具体实践过程中我们会发现这样子建设服务节点投入太大了。

所以我们需要具体分析，究竟哪些请求影响了最终用户体验，试图找到一个成本均衡点。

为了简化问题，现在的 CDN 服务一般会做静态资源的分发工作，实际上就是分离了服务的计算密集型和存储密集型这两种场景。

计算密集型的服务逻辑会比较复杂，节点部署起来会麻烦很多，而静态资源存储就简单很多了。


### 如何计算用户与节点的距离

用户在访问一个服务的时候会请求 DNS 服务，解析域名得到 IP 地址，再与该地址建立链接将数据包发送给主机实现通信。

![](https://segmentfault.com/img/remote/1460000019036402)

DNS 分为客户端和服务端，客户端将 name, type, class 打包成一个 questions 通过 UDP(DoT, DoH) 发送给 DNS 服务端,

DNS 服务端收到请求后如果有缓存就直接充当 Proxy 代理直接返回地址（递归查询），否则会进行 Resolve 解析（迭代查询），具体解析过程是这样的：

首先，会将域名进行拆解， 以 www.example.com 为例，它实际上是 www.example.com. 注意尾部的 `. (dot)` 域名规范中是有 dot 的，只是一般用户不会输入这个而已。

按 `. (dot)` 分割得到： [ 'www', 'example', 'com' ] 开始迭代查询：

1. 向 DNS Root Name Server[1] 查询 .com 的 NS 记录


```bash
~$ dig com NS @a.root-servers.net

; <<>> DiG 9.10.6 <<>> com NS @a.root-servers.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63564
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 13, ADDITIONAL: 27
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;com.				IN	NS

;; AUTHORITY SECTION:
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
```

2. 向 .com 的 TLD Name Server 查询 example 的记录

```bash
~$ dig example.com NS @e.gtld-servers.net.

; <<>> DiG 9.10.6 <<>> example.com NS @e.gtld-servers.net.
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19363
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 2, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;example.com.			IN	NS

;; AUTHORITY SECTION:
example.com.		172800	IN	NS	a.iana-servers.net.
example.com.		172800	IN	NS	b.iana-servers.net.
```

3. 以此类推，直到拿到最后一个结果为止，需要注意的是除了最后一个查询中间的查询都是 NS 记录。

对于 CDN 网络而言，通常是站点将域名通过 CNAME 记录指向到 CDN 服务商的域名上，至于为什么要用 CNAME 而不是 A 记录，这样做的好处是服务商可以控制 CDN 解析过程，而不必修改站点的 DNS 设置。

由于 CNAME 到了 CDN 服务商的域名，CDN 服务商通常会自行搭建 DNS 服务来获取用户的 DNS Question 和 IP 地址来智能返回距离用户最近的 CDN 节点 IP 地址 DNS Answer。

### CDN 节点如何工作？

首先经过 L4 四层交换机进行 NAT/SNAT 数据转发将流量分发给 Edge Server 边缘服务器，边缘服务器内部再经过 L7 七层交换机进行请求内容识别。

上面说的 L4/L7 指的是 OSI Layers Model 的层级：

TODO：补充一张 Wikipedia 中 OSI 的图

L4 协议的工具一般可以使用 F5 Network 和 Linux Virtual Server

通过NAT实现局域网IP和公网IP（VIP）的映射关系。

L7 协议可以使用 HAProxy、Nginx 等工具实现反向代理和负载均衡：

CDN 按照层级划分，会分为节点服务和中心服务，节点服务如果存在缓存资源就会直接返回给用户，如果不存在资源就会访问中心服务（二级缓存），然后中心服务会执行 回源 动作并缓存。