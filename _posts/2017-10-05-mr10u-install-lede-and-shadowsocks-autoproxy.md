---
layout: post
title: 基于TP-Link TL-MR10U的自动代理路由器
---

## 前言
[TP-Link TL-MR10U](http://www.tp-link.com.cn/product_300.html)是一款便携路由器，参数如下：
* 内置一个2600mAh的18650电芯;
* CPU AR9331@400MHz, Ram 32MiB, Flash 4MiB;
* 1 x 10/100Mbit RJ45以太网接口，b/g/n 1T1R 150Mbps无线；
* 1x USB 2.0 microUSB充电接口(5V 1A)，Reset键，1 x LED指示灯，电源开关。

具体硬件参数见[https://wiki.openwrt.org/toh/tp-link/tl-mr10u](https://wiki.openwrt.org/toh/tp-link/tl-mr10u)

根据LEDE Project的[说明](https://lede-project.org/supported_devices)
>Devices with ≤4MB flash and/or ≤32MB ram suffer from limitations in extensibility and stability of operation.

因此，在安装[LEDE](https://lede-project.org/)系统及[Shadowsocks](https://github.com/shadowsocks/openwrt-shadowsocks)之前，需要对TL-MR10U升级改造。

Flash芯片和内存芯片均为表贴封装(SMT)，可以自己动手，换Flash还算比较简单(虽然本人也搞砸了，千万要注意防静电，万一芯片被静电击穿，就报废了)，内存芯片密密麻麻的引脚看着就头大，只好求助万能的某宝。

[无线飞翔](https://shop103150354.taobao.com/)手艺不错，价格当然也够败家，升级顶配16MiB闪存、64MiB内存大概50￥左右。

## 步骤
### 安装LEDE
[LEDE Project](https://lede-project.org/)与[OpenWrt](https://openwrt.org/)的关系在此就不多说了，后者的最后一版是Chaos Calmer 15.05.1，而前者已经是LEDE 17.01.3。

首页点[Download Firmware](https://downloads.lede-project.org/releases/17.01.3/targets/), ar71xx, generic, tl-mr10u-v1-squashfs-factory.bin(从官方固件安装)，注意做好文件校验。

在官方固件的Web管理界面选择固件升级，不出意外的话，路由器重启之后就进入了LEDE系统。

### 方案选择
[Shadowsocks](https://shadowsocks.org/en/index.html)是一个socks5代理软件，支持多种客户端，我们使用的是[libev](https://github.com/shadowsocks/shadowsocks-libev)版本。

此外，为了解决DNS污染问题，还需要[openwrt-dns-forwarder](https://github.com/aa65535/openwrt-dns-forwarder)将DNS请求的UDP包通过Shadowsocks提供的TCP加密通道中转。

DNS服务器需要选择Google的`8.8.8.8/8.8.4.4`或者OpenDNS的`208.67.222.222/208.67.220.220`这种自身数据无污染的recursive DNS服务器。

中国境内(GFW内)的recursive DNS服务器，如运营商提供的或者阿里巴巴提供的均遭到了GFW的污染。

因为recursive DNS服务器想当于替用户(DNS请求，如twitter.com的IP地址是？)寻找答案，如果自身没有该域名的缓存(公共的recursive DNS服务器一般缓存了大量结果，国内的服务器对被GFW污染的域名的缓存结果当然是被污染的)，则从根服务器开始，到.com域名服务器，到twitter.com域名服务器。

请求是明文的UDP包，到了向twitter.com域名服务器发起询问请求时，GFW立刻回复一个错误的结果(一般是IP地址错误)。

所以，为了获取twitter.com这类被GFW污染域名的正确解析，openwrt-dns-forwarder需要使用Google Public DNS这种境外的DNS服务器。

应对GFW DNS污染的常见方法如下：
* DNS over TCP
* 非标准端口DNS服务器
* DNS over HTTP
* DNSCrypt

HTTP数据在传输层使用TCP(不考虑QUIC这类)，因此DNS over HTTP本质也是DNS over TCP。

GFW对DNS请求的污染目前仅限于常规端口(53)的常规数据包(UDP)，因此，为了避免污染，可以选择TCP包的DNS请求或者非常规端口的境外DNS服务器(不能使用国内常规的公共DNS，因为它的缓存或请求都是被污染的)。

路由器平台的TCP DNS方案有：
* [DNS-Forwarder for OpenWrt](https://github.com/aa65535/openwrt-dns-forwarder)，不仅使用Shadowsocks的TCP，而且是加密的，利用已有的Shadowsocks服务，资源开销小；
* [pdnsd](http://members.home.nl/p.a.rombouts/pdnsd/)，项目久未更新，最新的版本是2012-03-17发的，简单用TCP封装了DNS的UDP数据包；
* [Unbound](https://www.unbound.net/)，原理同上。

Shadowsocks的ss-tunnel是把DNS的UDP包加密，但仍然是UDP包，提供了DNS客户端与服务器间的一个加密信道，在一些UDP不稳定的环境下效果较差。

DNSCrypt将DNS请求用SSL封装(使用443端口)，支持TCP/UDP。

如果对国内网站，如淘宝的DNS请求也使用境外DNS服务器解析，则解析结果速度可能比较慢(解析成淘宝国际版)，可以使用[ChinaDNS](https://github.com/shadowsocks/ChinaDNS)，路由器可以使用[OpenWrt版](https://github.com/aa65535/openwrt-chinadns)，具体设置见项目的README.md，与DNS Forwarder配合的[wiki](https://github.com/aa65535/openwrt-chinadns/wiki/Use-DNS-Forwarder)。

本方案用到的项目有：
* [DNS Forwarder](https://github.com/aa65535/openwrt-dns-forwarder)
* [Shadowsocks-libev for OpenWrt](https://github.com/shadowsocks/openwrt-shadowsocks)
* [OpenWrt LuCI for Shadowsocks-libev](https://github.com/shadowsocks/luci-app-shadowsocks)
* [luci-app-dns-forwarder](https://github.com/aa65535/openwrt-dist-luci)(可选)

### 安装配置
#### WAN口设置
__此设置仅针对只有一个以太网口的设备，如TL-MR10U__

在LEDE的Web控制台，Network, Interfaces, LAN, edit, Physical Settings，Interface下取消对以太网口Ethernet Adapter: "eth0"的勾选，Save & Apply。

回到Interfaces, add new interface, Name of the new interface可以填`wan`, Protocol of the new interface选`DHCP Client`即可，Cover the following interface选Ethernet Adapter: "eth0", Submit.

这样以太网口就设置成了DHCP Client，可以连接上级路由器，然后设置无线，之后的操作，笔记本通过无线连接TL-MR10U，TL-MR10U通过以太网口连接互联网。

#### 安装软件
##### 1. 更新软件源
```
opkg update
```
##### 2. 添加软件源
添加[OpenWrt-dist](http://openwrt-dist.sourceforge.net/)的源
```
wget http://openwrt-dist.sourceforge.net/packages/openwrt-dist.pub
opkg-key add openwrt-dist.pub
```
获取路由器架构
```
opkg print-architecture | awk '{print $2}'
```
添加以下至`/etc/opkg/customfeeds.conf`
```
src/gz openwrt_dist http://openwrt-dist.sourceforge.net/packages/LEDE/base/{architecture}
src/gz openwrt_dist_luci http://openwrt-dist.sourceforge.net/packages/LEDE/luci
```
记得把`{architecture}`替换成自己路由器的架构(不含花括号)，例如TL-MR10U使用的AR9331 CPU，架构是`mips_24kc`。

##### 3. 安装软件
```
opkg update
opkg install dns-forwarder
opkg install luci-app-dns-forwarder
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
```
##### Notes:
这样做的好处是，无需分别下载所需的各个软件包，还可以及时发现更新，命令如下：
```
opkg update
opkg list-upgradable
```
当然，你也可以分别下载所需软件包，见各个项目的releases。

#### 配置DNS-Forwarder
打开LEDE的Web控制界面，因为我们已经安装了`luci-app-dns-forwarder`和`luci-app-shadowsocks`，所以导航栏的Status, System后面出现了Services，其中包括ShadowSocks和DNS-Forwarder两个链接。

![Luci Services](/images/luci_services.png "Luci Services")

点击`DNS-Forwarder`，勾选`Enable`,Save & Apply。

![DNS-Forwarder](/images/dns_forwarder.png "DNS-Forwarder")

#### 配置ShadowSocks

点击`ShadowSocks`, `Servers Manage`, `Add`，添加你的Shadowsocks服务器

![添加Shadowsocks服务器](/images/shadowsocks_servers_manage.png "添加Shadowsocks服务器")

* Alias是此服务器的别称，可选，待会用到；
* TCP Fast Open如果服务器支持该特性，勾选；
* Server Address是服务器的IP；
* Server Port是服务器的端口；
* Connection Timeout可以使用默认值，与性能有关，可以自己搜索，研究；
* Password是服务器的密码；
* Directly Key普通设置的服务器无需填写；
* Encrypt Method根据服务器的设置选择；
* Plugin等可以自行研究。

ShadowSocks - Access Control通常只需设置`Zone WAN`下的地一个条目`Bypassed IP List`，顾名思义，这是一个[CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)格式的以行分隔的IP列表文件，里面是直连的IP，其余的IP全部走ShadowSocks代理。
```
1.0.1.0/24
1.0.2.0/23
1.0.8.0/21
```
可通过以下命令获取
```
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' |     awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /tmp/chinadns_chnroute.txt
```
确认文件格式无误后，通常放在`/etc/chinadns_chnroute.txt`。
然后，Save & Apply。

* 最后，打开ShadowSocks - General Settings，在Transparent Proxy中，Main Server下拉选择服务器，此时看到的是服务器的Alias。如果路由器是多核心，可以添加与核心数量对应的服务器，提高代理性能；如果此处选择的服务器不同，则起到负载均衡的作用，不过某些网站可能出现异常。

* UDP-Relay Server与游戏相关，ShadowSocks默认只代理TCP(ss-tunnel除外)，设置此处，可以让UDP也走代理。此外，有些软件/服务(猜测某些Google服务)不使用设备网络默认的DNS，设置此处也可以解决这个问题；

* Local Port, Override MTU使用默认值即可；

SOCKS5 Proxy提供了一个Socks5代理，开启后，浏览器如Chrome使用[Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)设置代理类型Socks5，IP地址路由器的IP，端口默认的1080，即可实现代理。

Port Forward即ss-tunnel功能，提供一个监听在本地某端口(默认5300)的无污染DNS服务。

Save & Apply，不出意外，自动代理的Shadowsocks部分就完成了，Shadowsocks和DNS-Forwarder启用后，默认开机自启，如果出了问题，通常是启动的问题，根据提示排错即可。
```
ps | grep ss-redir
ps | grep dns-forwarder
```
查看两个服务是否启动。

#### 配置Dnsmasq
编辑`/etc/dnsmasq.conf`
```
cache-size = 4096
min-cache-ttl = 3600
no-resolv
no-poll
domain-needed
server = 127.0.0.1#5300
```

重启Dnsmasq
```
/etc/init.d/dnsmasq
```
至此，路由器就实现了自动代理。

* cache-size设置本地缓存的DNS查询结果数量，根据路由器内存大小和网络用户数量确定
```
killall -s USR1 dnsmasq
logread
```
```
cache size 4096, 0/0 cache insertions re-used unexpired cache entries.
```
如果`/`左边的数量过大，说明缓存的DNS数量不够多，大量的缓存在过期前被丢掉，cache-size不够大；

* min-cache-ttl强行修改DNS请求的结果中的TTL(Time To Live)，即有效时间，单位S秒。

有些大型服务器，为了负载均衡，TTL通常设置的比较小(几十秒)，但通过DNS-Forwarder TCP方式解析DNS，时间开销略大，通常与服务器的Ping时长有关，而且与常规DNS服务器相比，解析失败的可能性较大，故此将min-cache-ttl尽量设置大一些，提高DNS解析速度，增强体验。

该选项是最近版本才加入的，[Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)作者建议慎重使用：
>Add --min-cache-ttl option. I've resisted this for a long time, on the grounds that disbelieving TTLs is never a good idea, but I've been persuaded that there are sometimes reasons to do it. (Step forward, GFW). To avoid misuse, there's a hard limit on the TTL floor of one hour. Thanks to RinSatsuki for the patch.

如上所说，由于作者限制，LEDE源中的Dnsmasq min-cache-ttl最大值是3600(一小时)，大于3600的值按照3600处理，如果你有特殊需求，可以自己编译；

* no-resolv告诉Dnsmasq不使用运营商提供的DNS服务器(因为污染问题)；
>Don't read /etc/resolv.conf. Get upstream servers only from the command line or the dnsmasq configuration file.

* no-poll告诉Dnsmasq不去查看运营商提供的DNS服务器是否有变化；
>Don't poll /etc/resolv.conf for changes.

* domain-needed告诉Dnsmasq在发送DNS请求前，确认待查询域名有效；
>Tells dnsmasq to never forward A or AAAA queries for plain names, without dots or domain parts, to upstream nameservers. If the name is not known from /etc/hosts or DHCP then a "not found" answer is returned.

* server = 127.0.0.1#5300告诉Dnsmasq使用DNS-Forwarder提供的DNS解析服务，`#`后面是端口，你也可以使用ss-tunnel，[DNSCrypt](https://wiki.openwrt.org/inbox/dnscrypt)等服务。

总之，路由器使用的上游DNS服务器通过server设置，你也可以添加多个无污染的DNS服务，并添加`all-servers`设置，这样Dnsmasq会同时向所有的server发起请求，并使用返回最快的那个结果；

其他配置，见[Dnsmasq manpage](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html)。

#### Notes
国内也有些无污染的DNS服务器，如[中科大LUG](https://github.com/ustclug/neatdns)：
```
202.141.178.13(移动)
202.141.162.123(电信)
202.38.93.153(教育网)
202.38.93.94(教育网，备选)
```
USTC LUG无污染的实现方式也是通过VPN查询，备选DNS服务器的硬件、线路与主服务器物理隔离，提高可靠性。

此外USTC LUG的这些DNS服务器还提供TCP，DNSCrypt查询方式，详情见其相关[黑板报](https://web.archive.org/web/20160323000452/https://servers.ustclug.org/category/dns/)。

由于[USTC LUG黑板报](https://servers.ustclug.org/)与无污染DNS相关的通知仅中科大校内可见，所以，校外用户需要加入[Neat DNS Notification Google Group邮件列表](https://groups.google.com/forum/#!forum/neat-dns)来获取最新信息(私密群组，需登录Google帐号查看)。

[清华Tuna](https://tuna.moe/help/dns/):
```
101.6.6.6
2001:da8::666
```
## 后记
至此，你已经成功配置了一个自动代理的路由器，拥抱迟到的自由吧！

我不会为GFW做任何辩护，分享这篇文章只是希望帮助更多的人减少浪费在代理上的无意义的时间开销，把时间花在有价值的事情上。

China's first foray into global cyberspace was an email (not TCP/IP based and thus technically not Internet) sent on 20 September 1987 to University of Karlsruhe. It said:
>Across the Great Wall, we can reach every corner in the world.

>越过长城，走向世界。

而如今，GFW对互联网的封锁与干扰愈演愈烈，打脸如此，怡笑大方！

__P.S. 安全起见，建议购买[VPS](https://en.wikipedia.org/wiki/Virtual_private_server)自己搭建Shadowsocks代理服务器。
博主在用[Digital Ocean](https://www.digitalocean.com/?refcode=de1b31930850)，最低配置5$/月，从[此链接](https://www.digitalocean.com/?refcode=de1b31930850)注册，账户充值5$后可以获得额外10$。__

## 鸣谢
[clowwindy](https://github.com/clowwindy)

[Jian Chang](https://github.com/aa65535)

[飞羽博客博主cokebar](https://cokebar.info/about/)

## 参考
* [https://cokebar.info/archives/664](https://cokebar.info/archives/664)
* [http://openwrt-dist.sourceforge.net/](http://openwrt-dist.sourceforge.net/)
