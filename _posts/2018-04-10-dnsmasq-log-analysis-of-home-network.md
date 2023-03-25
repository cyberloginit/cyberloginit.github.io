---
layout: post
title: Dnsmasq Log Analysis of Home Network
published: false
---

# Dnsmasq Log Analysis of Home Network

## 环境
天朝特殊的网络环境下，DNS解析优化是个很麻烦的问题。自己的OpenWrt路由器目前使用Shadowsocks-libev代理，采用国外IP全部走代理的方案。

DNS方面，保证解析结果正确(主要是免遭GFW污染)的同时，还要尽量优化解析结果对代理和直连网络的[CDN](https://en.wikipedia.org/wiki/Content_delivery_network)亲和性。

为了优化解析结果对代理的CDN亲和性，需要解析结果相对代理的位置/网络尽量是理想的。例如，如果你的代理服务器在Digital Ocean的San Francisco节点，对 google.com DNS解析的理想结果(IP)应该是Google服务器的Mountain View节点，而不是台湾节点。这样才能尽量达到低延时、高带宽的效果，虽然瓶颈往往在出国线路而不是大水管的代理和Google之间。

所谓直连网络，即国内的网站。考虑到国内运营商奇葩的peering策略，对于QQ，淘宝等服务尽量还是使用运营商的DNS。

## 方案
OpenWrt默认的DNS及DHCP工具是[Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html).

自己过去一直认为，针对上述问题，目前还没有优雅的解决方案。

某天读一篇不相关的文章时，意识到无论是国内域名白名单，即主DNS服务器使用对代理的CDN亲和的如[dns-forwarder](https://github.com/aa65535/openwrt-dns-forwarder)方案
```
server = 127.0.0.1#5300
server=/qq.com/{运营商DNS}
...
```
还是国外域名白名单
```
server = {运营商DNS}
server=/google.com/127.0.0.1#5300
...
```
都忽略了一个很重要的因素：自己的需求。

例如，自己很早就彻底注销了Facebook帐户，所以在国外域名白名单方案中加上Facebook的一些域名没有意义；

又如，国内的互联网服务，自己只使用QQ，微信，支付宝，同理，在国内域名白名单方案中加上百度或者外卖类的域名也纯属浪费路由器资源。

基于自己很少使用国内网络服务的现实，选择国内域名白名单的方案。

网上已经有很多这类项目，如[gfw_whitelist](https://github.com/breakwa11/gfw_whitelist)。

然而问题是，如何从诸多域名中剥离出自己会用到的？

显而易见的方案是，先添加自己能想到的域名，如QQ，支付宝等。

但此方案的弊端很明显：
1. 需要经常ssh进路由器修改dnsmasq.conf文件
2. 获取相关域名或CDN域名比较麻烦
3. 很难判断某个域名的解析结果对服务(如QQ)体验产生了多大影响

至此，终于引入了本文的正题：使用Dnsmasq的日志记录功能，分析自己的使用场景下用到的域名。

## Dnsmasq Log
在`/etc/dnsmasq.conf`中加入
```
log-queries = extra
log-facility = /etc/dnsmasq.log
```
开启dnsmasq的详细日志记录功能。

`log-queries = extra`表示记录详细的日志，`log-facility = /etc/dnsmasq.log`指定了存储日志的文件。

注意：如果路由器存储空间有限，可以把`log-facility`设在挂载的U盘里，每天产生的日志可达1MB以上。

## 分析脚本
考虑到扩展性，决定使用Python写分析脚本(其实是因为Shell忘光了)。
```
#!/usr/bin/env python3

import collections

# logfile
DNSLOG = "dnsmasq.log"

domain_list = []
with open(DNSLOG, "r") as logfile:
    for query in logfile:
        try:
            # 26446 in "dnsmasq[26446]:" may be different
            trunk = query.split("dnsmasq[26446]:")[1]
            action = trunk.split(" ")[3]
            domain = trunk.split(" ")[4]
            # skip non domain
            if (domain.split('.')[1].strip() == 'lan') or (len(domain.split('.')) == 1):
                continue
            if action.strip() == 'query[A]':
                # uncomment it will change to top domain only
                # domain = domain.split('.')[-2] + '.' + domain.split('.')[-1]
                domain_list.append(domain)
        except:
            continue

counter = collections.Counter(domain_list)
top_20 = counter.most_common(20)

for d in top_20:
    print(d)
```
脚本原理很简单，读取日志文件的每一行，拆分得到action(query[A], query[AAAA], cached ...)和domain，

日志中还有其他类型的行，不符合我们的拆分条件，自然抛出异常，直接continue(忽略)；

然后过滤掉格式非法的域名，之后只记录(添加到列表)解析域名的IPv4地址的请求(query[A])；

如果有需要，可以只记录一级域名，毕竟我们的白名单里面只需指定一级域名，即可覆盖其下的所有子域名。

最后使用collections里面的频率计算算法(自己造轮子不知道得跑到什么时候)，打印出top 20的域名。
```
('microsoft.com', 1734)
('google.com', 1377)
('googleapis.com', 1328)
('windows.com', 517)
('live.com', 492)
('qq.com', 351)
('gstatic.com', 342)
('msedge.net', 291)
('hotmail.com', 282)
('spotify.com', 253)
('skype.com', 210)
('live.net', 167)
('windows.net', 163)
('doubleclick.net', 158)
('githubusercontent.com', 148)
('twitter.com', 126)
('googleusercontent.com', 117)
('scdn.co', 109)
('googlesyndication.com', 99)
```
以上是自己大概一周的流量top 20。

Google的域名诸多完全在预料之中，但巨硬的域名那么多，请求频率那么高什么鬼？毕竟只有一台Windows笔记本，Skype更是完全没用过...

`doubleclick.net`频率也那么高，不屏蔽掉说不过去了。

至此，自己可以高效的获得添加到白名单里的国内域名了。

## Reference

* [https://stackoverflow.com/questions/2161752/how-to-count-the-frequency-of-the-elements-in-a-list](https://stackoverflow.com/questions/2161752/how-to-count-the-frequency-of-the-elements-in-a-list)
* [https://github.com/lennylxx/ipv6-hosts/wiki/How-to-speed-up](https://github.com/lennylxx/ipv6-hosts/wiki/How-to-speed-up)
