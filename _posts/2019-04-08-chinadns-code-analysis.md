---
layout: post
title: ChinaDNS原理与源码分析
published: false
---

# ChinaDNS原理与源码分析

## 背景
[ChinaDNS](https://github.com/shadowsocks/ChinaDNS/) 是 [Shadowsocks](https://github.com/shadowsocks/shadowsocks)作者 [clowwindy](https://github.com/clowwindy) 开发的一个中国特色防污染DNS解析工具。

### 递归DNS服务器
通常，国内运营商提供的DNS服务器或国内公共的如114这种DNS服务器对国内网站的解析结果对国内用户是最优的，这里所说的DNS服务器指递归Recursive DNS，其工作原理如下：
> You might have been able to guess what a recursive DNS server does by its name—it recurses, which means that it refers back to itself.

1. 用户向递归DNS服务器发起解析请求，递归DNS首先查询缓存，如果有对应结果就直接返回给用户；如果没有，就先询问根域名服务器 root-servers.net；

> Operators who manage a DNS recursive resolver typically need to configure a “root hints file”. This file contains the names and IP addresses of the root servers, so the software can bootstrap the DNS resolution process. For many pieces of software, this list comes built into the software.

2. 根域名服务器只负责'.'的解析，根据请求解析的域名返回对应的国家顶级域名（如 .cn对应 dns.cn）或通用顶级域名（如 .com对应 gtld-servers.net）的权威域名服务器IP；
3. 国家根域名服务器dns.cn或通用根域名服务器gtld-servers.net根据请求解析的域名返回该域名对应的权威域名服务器IP。

下面分别举例分析国家域名和顶级域名的解析过程。

#### 国家域名
国家域名的解析过程以工信部官网www.miit.gov.cn为例：

```
[~]$ dig www.miit.gov.cn @114.114.114.114 +trace

; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> www.miit.gov.cn @114.114.114.114 +trace
;; global options: +cmd
.                       439144  IN      NS      i.root-servers.net.
.                       439144  IN      NS      d.root-servers.net.
.                       439144  IN      NS      b.root-servers.net.
.                       439144  IN      NS      k.root-servers.net.
.                       439144  IN      NS      g.root-servers.net.
.                       439144  IN      NS      c.root-servers.net.
.                       439144  IN      NS      a.root-servers.net.
.                       439144  IN      NS      f.root-servers.net.
.                       439144  IN      NS      l.root-servers.net.
.                       439144  IN      NS      e.root-servers.net.
.                       439144  IN      NS      j.root-servers.net.
.                       439144  IN      NS      m.root-servers.net.
.                       439144  IN      NS      h.root-servers.net.
;; Received 239 bytes from 114.114.114.114#53(114.114.114.114) in 24 ms

cn.                     172800  IN      NS      g.dns.cn.
cn.                     172800  IN      NS      b.dns.cn.
cn.                     172800  IN      NS      f.dns.cn.
cn.                     172800  IN      NS      c.dns.cn.
cn.                     172800  IN      NS      ns.cernet.net.
cn.                     172800  IN      NS      a.dns.cn.
cn.                     172800  IN      NS      d.dns.cn.
cn.                     172800  IN      NS      e.dns.cn.
;; Received 734 bytes from 192.112.36.4#53(g.root-servers.net) in 246 ms

miit.gov.cn.            86400   IN      NS      dns.miit.gov.cn.
;; Received 328 bytes from 203.119.28.1#53(d.dns.cn) in 240 ms

www.miit.gov.cn.        86400   IN      CNAME   www.nsae.miit.gov.cn.
nsae.miit.gov.cn.       86400   IN      NS      ns1.nsae.miit.gov.cn.
nsae.miit.gov.cn.       86400   IN      NS      ns2.nsae.miit.gov.cn.
;; Received 135 bytes from 202.106.120.1#53(dns.miit.gov.cn) in 41 ms

```

可以看到对工信部官网www.miit.gov.cn解析的整个流程包括root-servers.net->dns.cn->dns.miit.gov.cn。

#### 通用域名
通用域名的解析过程以dl.google.com为例：

```
[~]$ dig dl.google.com @114.114.114.114 +trace

; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> dl.google.com @114.114.114.114 +trace
;; global options: +cmd
.                       172510  IN      NS      d.root-servers.net.
.                       172510  IN      NS      a.root-servers.net.
.                       172510  IN      NS      l.root-servers.net.
.                       172510  IN      NS      k.root-servers.net.
.                       172510  IN      NS      e.root-servers.net.
.                       172510  IN      NS      m.root-servers.net.
.                       172510  IN      NS      g.root-servers.net.
.                       172510  IN      NS      b.root-servers.net.
.                       172510  IN      NS      i.root-servers.net.
.                       172510  IN      NS      h.root-servers.net.
.                       172510  IN      NS      f.root-servers.net.
.                       172510  IN      NS      j.root-servers.net.
.                       172510  IN      NS      c.root-servers.net.
;; Received 239 bytes from 114.114.114.114#53(114.114.114.114) in 24 ms

com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      m.gtld-servers.net.
;; Received 1173 bytes from 199.7.91.13#53(d.root-servers.net) in 203 ms

google.com.             172800  IN      NS      ns2.google.com.
google.com.             172800  IN      NS      ns1.google.com.
google.com.             172800  IN      NS      ns3.google.com.
google.com.             172800  IN      NS      ns4.google.com.
;; Received 775 bytes from 192.43.172.30#53(i.gtld-servers.net) in 236 ms

dl.google.com.          604800  IN      CNAME   dl.l.google.com.
dl.l.google.com.        300     IN      A       172.217.6.46
;; Received 77 bytes from 216.239.34.10#53(ns2.google.com) in 215 ms
```

可以看到对dl.google.com解析的整个流程包括root-servers.net->gtld-servers.net->ns.google.com。

### 权威DNS服务器
上述提到的 root-servers.net, dns.cn, gtld-servers.net, dns.miit.gov.cn, ns.google.com都属于权威DNS服务器，他们近对自己负责的如., .cn, .com, miit.gov.cn, google.com等域名的子域名提供服务，
而递归DNS服务器则代替用户从这些权威DNS服务器处查询某个具体域名（如dl.google.com）对应的IP地址。

接下来，我们从IP和域名两个角度介绍“国内网站”的概念。

### 国内网站
国内网站包括网站的IP地址和域名两部分，这里“国内”的判断依据是是否受中国（不包括港澳台）法律管辖。

#### 国内IP
IP在中国的意思是这个IP或IP段是属于某个中国AS [Autonomous System](https://en.wikipedia.org/wiki/Autonomous_system_(Internet))，该AS使用BGP及相关协议将这个IP或IP段的信息传播到整个互联网。

正常情况下，公司或运营商从其所在国家所属的RIR [Regional Internet Registry](https://en.wikipedia.org/wiki/Regional_Internet_registry) 申请IP段，使用期间要缴纳管理费用。
但目前IPv4地址已经分配完了，一些公司可以从其他公司或运营商处“购买”IP段，导致IPv4的地址比较混乱。
例如，一个中国公司可以从日本公司买IP段，然后借助中国电信的AS连接到互联网，这段IP就变成了“国内IP”。

国内网站就是其IP使用中国的AS连接国际互联网的网站。

#### 国内域名
首先，后缀为.cn的域名都是国内域名；其次，对应IP在中国的域名也是国内域名，因为这些IP对应的服务器受中国法律管辖。

根据中国的《互联网信息服务管理办法》，
> 国家对经营性互联网信息服务实行许可制度；对非经营性互联网信息服务实行备案制度

因此可以确定只要是“中国网站”理论上都要备案，即使使用非.cn域名，如 taobao.com。

绝大部分国内域名对应的IP都在中国，大部分国外域名对应的IP都不在中国。比较麻烦的是跨国公司和CDN，例如Apple是少数在中国正常开展业务的国外IT公司，taobao.com也有对应的海外版，
它们的权威DNS服务器可能对部分子域名使用了ECS [EDNS Client Subnet](https://en.wikipedia.org/wiki/EDNS_Client_Subnet) 技术，即根据请求来源IP所在位置返回位置最近（最优）的结果，对中国的用户自然返回国内IP。


### 中国特色的DNS解析问题
由于中国的互联网国际出口带宽小，且GFW对加密的未知连接限速甚至重置，因此一些人选择国外IP全部走代理，国内IP直连的方案，解决了无法访问某些IP的问题。

考虑到GFW会通过抢答的方式污染某些域名的解析，即任何发往国外IP的DNS请求都会受到污染，即使该国外IP根本不是递归DNS服务器。

目前解决GFW造成的DNS污染的方法可以分为两类。

#### 穿墙解决DNS污染
穿墙即利用GFW污染DNS解析的具体实现中的弱点，直接获得无污染的DNS解析结果，本质是直连。

主要方法有以下三种：
1. 过滤掉过快返回的DNS响应（ChinaDNS）：因为被污染的域名都在国外，解析结果返回所需时间不可能少于某个阈值；
2. 在全部DNS请求包中构造指针（ChinaDNS）：GFW无法识别DNS请求中的一些指针压缩方式，但少数国外递归DNS服务器（如8.8.8.8）却可以；
3. 过滤掉GFW返回的固定IP：之前GFW返回的污染IP是固定的几十个国外IP；
4. TCP查询国外DNS：GFW之前只污染使用UDP的DNS查询；
5. 使用非标准端口查询国外DNS：GFW之前只污染对国外DNS服务器标准53端口的查询；

方法一基于时间的过滤实践证明不够稳定；方法二基于指针的方法，[这里](https://gist.github.com/klzgrad/f124065c0616022b65e5)提出的较早，[这里](https://blog.ddosolitary.org/posts/research-on-dns-packet-forgery-of-gfw)证明已经失效；方法三随着GFW返回的污染IP随机化完全失效；方法四目前似乎已经失效，GFW开始污染或reset DNS的TCP查询；方法五目前似乎可用。

#### 翻墙解决DNS污染
GFW不断升级导致穿墙解决DNS污染的难度巨增，因此目前广泛使用的是翻墙方法，本质是代理。

对于一些“非法”域名，如 twitter.com，GFW会污染发往任何国外IP的DNS请求，因此需要对该DNS请求加密。
但国外的递归DNS服务器通常无法识别这种私有的加密处理，因此需要代理在国外将该加密请求解密后发给国外的递归DNS服务器。

这种方式的优点是比较稳定，代理一般用国外的虚拟主机VPS，如 [Digital Ocean](https://www.digitalocean.com/?refcode=de1b31930850)，代理和国外DNS服务器之间的网络质量通常很好，
影响该方法稳定性的主要是用户与代理间的网络质量。

这种方式的缺点是引入的时间开销过大，影响用户体验，毕竟用户对DNS解析花费的时长比较敏感，这个问题可以通过在本地增加DNS缓存并修改增大DNS解析结果的TTL部分解决，
使用OpenWrt Dnsmasq的用户可以参考[这里](https://cyberloginit.com/2018/04/10/dnsmasq-log-analysis-of-home-network.html)。

### DNS解析优化
这里的优化包括减少解析的时间花销和获取靠近用户的IP两个发方面。

这里讨论的前提是使用代理解决GFW对某些域名的污染问题，且国外的IP全部走代理。

国内的域名都受中国的法律管辖，“违法”者直接拔网线即可，没有必要使用DNS污染，对于国内域名，我们希望使用国内的递归DNS服务器解析，一方面显著降低时间开销，另一方面，保证解析结果IP靠近用户。

国外的域名，由于我们设置国外的IP全部走代理，考虑到CDN亲和性（由于ECS技术），自然希望国外的域名全部使用代理服务器解析，这样解析结果IP和代理间的带宽和延迟都较好。
这里的假设是使用代理解析的国外域名对应的IP都在国外。

现在的问题是：我们如何判断一个域名属于国内还是国外？不同于IP地址，域名理论上是没有国别属性的，而且同一个域名也可能在多个国家开展业务。
对于IP地址的国别，我们可以使用一些组织提供的CIDR列表，如IPIP整理的[这份](https://github.com/17mon/china_ip_list)。
对于域名的国别，当然可以使用手动搜集域名名单的方式，但不够优雅。

clowwindy在ChinaDNS中采用的方法十分优秀，但网上对其工作原理和具体流程的介绍不仅语焉不详，有些还存在本质错误，下一章我们将分析C语言版ChinaDNS的源码，介绍其工作原理和具体流程。

## C版ChinaDNS源码分析
Shadowsocks作者clowwindy自从2015年8月喝茶后，也不再维护ChinaDNS，Jian Chang [aa65535](https://github.com/aa65535) fork了[一份](https://github.com/aa65535/ChinaDNS)，目前主要维护[OpenWrt版](https://github.com/aa65535/openwrt-chinadns)。

分析 [Repo]((https://github.com/aa65535/ChinaDNS)) 的commit记录可以看到，fork版新增了4个commit，主要是增加了可信DNS服务器选项。

C版ChinaDNS的源码在`ChinaDNS/src/`目录下，包括`chinadns.c`, `local_ns_parser.c`, `local_ns_parser.h`三个文件。

代码的流程显然在主文件`chinadns.c`中，我们从第178行的main函数入手，
201行的`while`循环应该是ChinaDNS工作时的主体部分，输入的DNS解析请求放入local_sock，输出的DNS解析结果放入remote_sock，分别由228行的`dns_handle_local()`和230行的`dns_handle_remote()`处理。


### dns_handle_local()
616行开始的`dns_handle_local()`会从`local_sock`读取解析请求到`msg`，然后进行格式化操作，包括为每个DNS解析请求设置ID并存到队列中，接下的`if`语句处理指针压缩选项，`for`语句遍历用户预设的若干DNS服务器，
`test_dns_server_type()`判断每个的DNS服务器的类别，即 `CHN_DNS, TRUSTED_DNS, FOREIGN_DNS` 三种，具体的判断方式包括比对chnroute_list，即中国IP的CIDR列表。

如果是TRUSTED_DNS，直接调用
```
send_request(dns_server_addrs[i], global_buf, len);
```
将 `global_buf` 中的DNS解析请求发送给该DNS服务器；

如果是FOREIGN_DNS，则需要使用DNS指针压缩技术避免GFW的污染
```
send_request(dns_server_addrs[i], compression_buf, len + 1);
```

### dns_handle_remote()
697行开始的`dns_handle_remote()`从`remote_sock`读取解析结果到`msg`，比较重要的代码如下：
```
r = should_filter_query(msg, src_addr);
if (r == 0) {
    if (verbose)
        printf("pass\n");
    if (-1 == sendto(local_sock, global_buf, len, 0, id_addr->addr, id_addr->addrlen))
        ERR("sendto");
} else if (r == -1) {
    schedule_delay(query_id, global_buf, len, id_addr->addr, id_addr->addrlen);
    if (verbose)
        printf("delay\n");
} else {
    if (verbose)
        printf("filter\n");
}
```
其中`should_filter_query()`函数就是ChinaDNS的点睛之笔，
```
static int should_filter_query(ns_msg msg, struct sockaddr *dns_addr) {}
```
该函数的输入参数`msg`是一个DNS解析结果，`*dns_addr`是返回该解析的DNS递归服务器的IP。

`dns_is_chn`表示返回某条解析结果的DNS服务器是国内的，`dns_is_foreign`则表示服务器是国外的。
```
int dns_is_chn = 0;
int dns_is_foreign = 0;
if (chnroute_file && (dns_servers_len > 1)) {
    dns_is_chn = (CHN_DNS == test_dns_server_type(dns_addr));
    dns_is_foreign = !dns_is_chn;
}
```
判断返回某个解析结果的DNS服务器是国内还是国外。
```
rrmax = ns_msg_count(msg, ns_s_an);
```
rrmax即一个DNS解析结果中Resource Record的数量。

818行的for循环中的以下代码就是ChinaDNS的工作原理所在。
```
      if (test_ip_in_list(*(struct in_addr *)rd, &chnroute_list)) {
        // result is chn
        if (dns_is_foreign) {
          if (bidirectional) {
            // filter DNS result from foreign dns if result is inside chn
            return 1;
          }
        }
      } else {
        // result is foreign
        if (dns_is_chn) {
          // filter DNS result from chn dns if result is outside chn
          return 1;
        }
      }
```


`test_ip_in_list()`判断解析结果IP是否为国内IP，如果是国内IP，且返回该DNS解析结果的服务器是国外的，并且开启了双向过滤，则`return 1`丢弃该解析结果，
对应737行的`r = should_filter_query(msg, src_addr);`且r不是0或-1的场景；

如果是国外IP，则如果返回该结果的DNS服务器是国内的，就`return 1`丢弃（过滤）该结果。

## 总结
分析源码后可以发现ChinaDNS工作的原则是尽量不感染DNS的正常解析流程，只根据规则过滤掉某些解析结果，对外的接口依然是普通的递归DNS。

收到用户发来的DNS解析请求后，分别向用户配置的国内和国外递归DNS服务器请求，然后等待返回结果，只要没有被过滤，先返回的解析结果被用户采用。

此外，过滤规则也很简单，不需要等国内和国外DNS服务器都返回结果后再做判断，正常情况下只丢弃国内DNS服务器返回国外IP的解析结果。

如果用户设置了双向过滤，则过滤掉国外DNS服务器返回国内IP的解析结果。对于国内域名，国内DNS理论上返回结果更快，用户根本不再等待国外DNS的结果；对于国外域名，国内DNS返回的结果（国外IP）直接被丢弃，
如果国外DNS返回了国内IP，这种情况比较罕见。

ChinaDNS的特点如下：

1. 充分利用了国内DNS服务器返回结果快的优势。例如，用户请求解析 taobao.com，则国内DNS服务器返回结果理论上比国外DNS快，用户此时无需等待直接使用国内DNS返回的结果，
同时避免了采用国外服务器解析结果情况下CDN亲和性不佳的问题；

2. 通过DNS服务器IP和DNS解析结果IP的国别判断解决了某些国外域名的DNS污染问题。例如，用户请求解析twitter.com时，国内DNS服务器依然更快返回结果（被污染的国外IP），
ChinaDNS根据规则丢弃国内DNS返回的国外IP，用户此时只需继续等待国外DNS返回未被污染的国外IP；

综上，ChinaDNS的工作原理并不像网上一些人说的是根据DNS服务器的国内国外（无污染）两个属性，和解析结果的国内国外两个属性，构成2x2的矩阵，然后分别对矩阵中的每个值做规则判断。

                                       | A record is local | A record is foreign
    local and poisoned dns server      |    a              |   b
    unpoisoned dns server              |    c              |   d

From the matrix, we get the result as follows,

ac: use local dns server result

ad: use local dns server result

bc: impossible. use unpoisoned dns server result

bd: use unpoisoned dns server result

上面是 [greendns](https://github.com/faicker/greendns) README 中的介绍，但ChinaDNS工作时并不会出现ac, ad, bc, bd这些情况，因为ChinaDNS不用等两个DNS服务器都返回结果后再根据该矩阵的规则做判断。

ChianDNS工作基于两个前提：
1. 国内DNS服务器返回结果比国外的服务器更快。如果国外DNS服务器返回结果更快（就会被采用），则ChianDNS丢失对国内域名的CDN亲和性；
2. GFW对国外域名的污染仅返回国外IP。如果GFW使用国内IP污染国外域名，即国内DNS服务器对twitter.com返回国内IP，由于ChinaDNS实际上无法区分twitter.com是国外域名，就不会丢弃该结果，GFW得逞；

__ChinaDNS工作原理其实是基于上述两个前提，根据一个规则“丢弃国内DNS服务器返回的国外IP解析结果”，达到防止DNS污染的同时，降低解析时间开销和增加CDN亲和性的目的。__

个人遇到过运营商劫持某些国外域名（如 ytimg.com, twitter.com）到国内IP的情况，此时ChianDNS无法取得防污染的效果。

总之，ChianDNS是个设计十分优秀的中国特色DNS优化方案，相比其他无污染DNS方案，在保证解析结果无污染的情况下有两个主要优势：
1. 降低了解析国内域名的时间开销；
2. 增强了对国内域名的CDN亲和性；

但该方案也存在不足。
1. 首先，其工作的第二个前提在有些运营商网络环境下已经不成立，虽然目前看应该是运营商DNS劫持而不是GFW的行为，但无法保证GFW后续不这样做，一旦GFW污染某些国外域名时随机返回国内或国外的IP，
ChianDNS将彻底失效；
2. 其次，对于少数仍在中国开展业务的国外互联网公司的域名解析，如果某个国外公司的服务器是国内IP，ChinaDNS也会返回该国内IP，例如微软、Apple等公司的某些服务。但我们不希望自己的数据进入这些公司在国内的服务器，一方面是安全隐私问题，另一方面是希望使用国外无阉割的完整版。例如，谷歌虽然已经“退出”中国，但其在中国的广告业务依然在开展，使用ChinaDNS，其某些域名会被解析到203.208开头的北京谷翔公司的服务器上，
但据说这些服务器并不稳定，这种问题，ChianDNS无法解决；

无污染DNS方案采用的方案本质可以分为两类：基于人工列表（域名方面有gfw黑白名单，IP方面有国内IP列表）的方式，基于理论设计的方式（域名方面有ChinaDNS，IP方面有[redsocks2](https://github.com/semigodking/redsocks)）。

为了解决ChinaDNS的第二点不足，我们可以修改国内IP列表，删除谷翔公司的IP段，此时国内DNS返回的203.208 IP被视为国外IP丢弃。

## Reference
* [https://github.com/shadowsocks/ChinaDNS](https://github.com/shadowsocks/ChinaDNS)
* [https://github.com/aa65535/ChinaDNS](https://github.com/aa65535/ChinaDNS)
* [https://github.com/aa65535/openwrt-chinadns](https://github.com/aa65535/openwrt-chinadns)
* [https://github.com/faicker/greendns](https://github.com/faicker/greendns)
* [https://umbrella.cisco.com/blog/2014/07/16/difference-authoritative-recursive-dns-nameservers/](https://umbrella.cisco.com/blog/2014/07/16/difference-authoritative-recursive-dns-nameservers/)
* [https://www.iana.org/domains/root/servers](https://www.iana.org/domains/root/servers)
* [https://gist.github.com/klzgrad/f124065c0616022b65e5](https://gist.github.com/klzgrad/f124065c0616022b65e5)
* [https://blog.ddosolitary.org/posts/research-on-dns-packet-forgery-of-gfw](https://blog.ddosolitary.org/posts/research-on-dns-packet-forgery-of-gfw)
* [https://cyberloginit.com/2018/04/10/dnsmasq-log-analysis-of-home-network.html](https://cyberloginit.com/2018/04/10/dnsmasq-log-analysis-of-home-network.html)
* [https://github.com/17mon/china_ip_list](https://github.com/17mon/china_ip_list)
