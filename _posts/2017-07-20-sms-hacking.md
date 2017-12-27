---
layout: post
title: SMS Hacking
---

## 前言
在即时聊天工具（IM）的冲击下，短信（SMS）作为一种聊天工具的功能被逐渐弱化，但其仍是一种相对正式的交流方式，并且是常用的双因子验证（Two Factor Authentication）渠道。
此外，当今智能手机依然向下兼容GSM协议栈的一些废弃功能。

本文探究静默短信（Silent SMS）、闪信（Flash SMS）的发送方式，Silent Ping的使用及这类短信在侦察中的使用。

## 环境
硬件：YYROBOT_SIM800开发板、中国移动SIM卡、两部智能手机、USRP B210

软件：screen、gr-gsm、Wireshark、[silent ping](https://github.com/itds-consulting/android-silent-ping-sms)

## SMS PDU
短信的收发有两种模式，PDU (Protocol Discription Unit) 和文本模式（Text mode），我们使用GSM终端（智能手机、功能手机、无线网卡等）发送短信时通常使用文本模式。某些无线网卡及支持AT指令的智能手机（三星的一些智能手机，如Galaxy S3）可使用PDU模式。Android原本提供sendRawPdu API，可惜最近的版本中已经移除。本次实验使用的SIM800开发板支持PDU模式，以下使用此开发板做演示。

1. 开发板连接电源，DB-9串口连接电脑，SMB口连接高增益GSM天线
2. 开发板上电，长按SIM800开关直至LED灯闪烁，开启SIM800模块
3. 电脑打开终端，连接开发板
```
sudo screen -L /dev/ttyUSB0 115200
```
4. 查看运营商、信号强度，确保已连接到网络
```
AT+COPS?
AT+CSQ
```
5. 切换短信至PDU模式，发送短信（静默短信、闪信、WAP PUSH等）
```
AT+CMGF=0
AT+CMGS=[PDU字节长度]<Enter>
>[PDU内容]<Ctrl+z>
```
PDU以`0001000B915121551532F400000CC8F79D9C07E54F61363B04`为例,
* 第一个字节`00`表示使用设备默认的SMSC号码；
* 第四个字节`0B`表示目标号码的长度为11（16进制）；
* 第五个字节`91`表示目标号码采用国际号码格式（+[国家代码，如美国是1,中国是86][手机号码]）；
* 接下来的几个字节编码方式有些奇怪，国际格式的目标手机号每两个数字为一组（不含+号，如果长度是奇数，最后补F），调换这两个数字的位置，
  如`5121551532F4`其实是个美国号码：(+1)5125551234；
* 接下来的一个字节是TP Protocol Identifier，`00`表示没有更高层的协议，`40`表示静默短信，`10`表示闪信；
* 其后的一个字节是TP Data Coding Scheme，表示短信内容的编码方式，`00`是默认的GSM 7编码，`C0`同样起到静默短信的效果；
* 接下来的一个字节是上文指定的编码方式下字符的长度（16进制表示）；
* 最后的这些字节是用16进制表示的编码后的内容，例如，若编码方式为GSM 7，则短信内容中的每个字符由7位二进制表示，但在PDU里面将这些二进制比特串以16进制显示，即每4比特显示为一个16进制数，具体规则可以参考[此文](http://mobiletidings.com/2009/02/11/more-on-the-sms-pdu/)

[此处](https://github.com/r00tb0x/SmsFuzzer)是一个开源的短信测试工具，支持多种短信，便于自动化。
## Silent Ping
Silent Ping利用了Android系统处理WAP PUSH时的一个“漏洞”，WAP PUSH是基于短信的一种应用，现在很少见，其效果是接收者的手机自动打开WAP PUSH内容中的URL链接，安全性较差。
虽然现代的系统不再支持该场景，直接忽略这种短信，但底层的基带处理器还是会按照普通短信的处理方式，向发送者返回deliver report，这一点可以被用来远程查询目标号码是否在线。
详情见[此文](https://www.evilsocket.net/2015/07/27/how-to-use-old-gsm-protocolsencodings-know-if-a-user-is-online-on-the-gsm-network-aka-pingsms-2-0/),
[这里](https://github.com/itds-consulting/android-silent-ping-sms)是一个开源实现。经测试，联通用户发到移动用户、电信用户的WAP PUSH无法收到deliver report，但移动用户可以正常查询联通用户、电信用户。

## 获得TMSI
为了保护用户隐私，避免用户被追踪，用户的IMSI值很少在GSM Um上直接传送，在一个小区(LAC)里面，每个用户有独特的TMSI值作为其代号，基站寻呼(Paging)用户时使用其TMSI值，
此外，[上行的通信请求，不管是发短信还是打通话，只能侦听到发起者的TMSI和接收者的MSISDN；而下行的通信，不管是来电话还是来短信，只能侦听到发起者的MSISDN和接收者的TMSI](https://cn0xroot.com/2016/12/09/gsm-hacking%EF%BC%9Athe-application-of-silent-sms-in-technical-investigation/)，为了对发起者/接收者进行监听，在知道其手机号的情况下还需要其在当前小区里的TMSI值，使用静默短信和Silent Ping均可以轻松实现：使用gr-gsm+USRP B210监听当前小区C0信道，大量发送静默短信或Silent Ping短信，筛选Paging Request，被呼叫次数超过我们设定阈值的TMSI便是我们呼叫的手机号对应的TMSI值。

## 总结
短信是一种古老的技术，其源于GSM标准，但目前GSM网络的安全性已岌岌可危，而且国内运营商（中国移动，中国联通）的GSM网络的语音、短信基本不加密，因此，重要的信息尽量不要使用短信传送。
此外，尽量不要使用手机号注册帐号，尤其是重要帐号不要使用短信作双因子认证。很多人没有意识到，一旦其手机号（短信）被攻破（通过鉴权伪装，短信监听等），其在互联网甚至整个社会的身份都会受到威胁。

## 参考资料
* <http://mobiletidings.com/2009/02/11/more-on-the-sms-pdu/>
* <http://mobiletidings.com/2009/02/12/sending-a-flash-sms-message/>
* <https://cn0xroot.com/2016/12/09/gsm-hacking%EF%BC%9Athe-application-of-silent-sms-in-technical-investigation/>
* <https://www.evilsocket.net/2015/07/27/how-to-use-old-gsm-protocolsencodings-know-if-a-user-is-online-on-the-gsm-network-aka-pingsms-2-0/>
* <https://cyberloginit.com/gr-gsm-notes/>
* <http://www.smartposition.nl/resources/sms_pdu.html>