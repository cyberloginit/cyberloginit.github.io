---
layout: post
title: gr-gsm notes
---

## 记录USRP B210, SIM800，开发板硬件配合gr-gsm, kalibrate, wireshark, screen等软件在Linux平台上的使用。

* 硬件：USRP B210, SIM800开发板
* 平台：Ubuntu 16.04 LTS 64位
* 软件：gr-gsm, kalibrate, wireshark, screen

1. gr-gsm[gr-gsm](https://github.com/ptrkrysik/gr-gsm)是目前使用最多的GSM网络分析工具，Airprobe已经好久没更新。
Ubuntu 16.04可以从PPA[gr-gsm PPA](https://launchpad.net/~ptrkrysik/+archive/ubuntu/gr-gsm
)安装，一方面节省了半小时左右的编译时间，另一方面，把头疼的依赖问题交给系统去解决，弊端是版本不够新，但本人使用中没有遇到太大问题。
 * grgsm_livemon监测给定频率，并将数据包传到localhost(loopback:lo)，可以使用wireshark直接查看
 * grgsm_capture可以抓取指定频率的GSM数据包并另存为本地文件，抓取60秒，文件大小为916M，不论该频率（信道）是否有通信进行，抓取的默认比特率为2Mb/s
 * grgsm_decode解码grgsm_capture抓取的数据，解码的默认速率为1Mb/s，因此需要手动添加 `-s 2000000` 参数

2. kalibrate[kalibrate](https://github.com/ttsou/kalibrate)提供GSM信号检测，时钟频率偏移计算功能。由于时钟精度对GSM网路而言至关重要，
建议有条件的为B210板子买个高精度的时钟源，如这款GPSDO[GPSDO](https://www.ettus.com/product/details/GPSDO-TCXO-MODULE)。
编译安装很简单：

```
./bootstrap
./configure
make
[sudo make install]
```
如果不安装，编译后的可执行文件在 `kalibrate/src` 目录下。

3. wireshark是很常用的工具了，基本上GSM相关的过滤规则以"gsm"开头，可用"gsmtap"过滤出GSM的数据包，"gsmtap && gsm_sms"可以查看监测频率GSM网络中传输的短信，
本人所在地区的中国移动，中国联通GSM网络短信均未加密。

4. screen是很好用的串口交互工具，用法很简单 `sudo screen [/dev/tty设备]` ，"- L"参数可以在命令执行的目录下生成与串口交互的日志文件。

5. 本人使用的SIM800开发板是国产YYROBOT_SIM800，支持两种电脑连接方式：TTL电平控制接口，兼容5V/3.3V，使用USB转TTL线即可连接电脑；通过DB9 RS232线或DB9转USB线连接。
这块板子使用SMA天线接口，但附送的天线太弱，室内基本很难连上网，最好换个高增益天线，SIM800模块默认不启动，开启开发板电源后，还要长按SIM800模块开关1秒以上启动。

6. AT指令是与SIM卡交互的关键，大小写不敏感，AT开头的输入才会被解读成指令执行
 * `sudo screen [/dev/tty设备]` 进入交互界面，一片空白，输入 `AT` ，回显 `ok`
 * 如果开启了PIN码保护，输入 `AT+CPIN=[PIN码]`
 * 查看信号强度 `AT+CSQ`
 * 查看连接的运营商 `AT+COPS?` , 如果没有显示运营商名字，说明没有连接到网络，开发板指示灯闪烁会比较急促
 * 开启工程模式 `AT+CENG=1,1`, 查看当前网络ARFCN及相关参数 `AT+CENG?`
 * 网络扫描 `AT+CNETSCAN`，同样获取ARFCN，可选
 * 锁定ARFCN `AT*CELLLOCK=1,1,[ARFCN]` , 方便之后抓包
 * 获取当前Kc值 `AT+CRSM=176,20256,0,0,9` ，去掉后两个字符
 * 获取当前TMSI `AT+CRSM=176,28542,0,0,11`，取前8个字符
 * 打电话 `ATD[号码;]`
 * 开启来电显示 `AT+CLIP=1`
 * 收到来电时，串口输出"Ring"字符串和来电号码等信息，输入 `ATA` 即可接电话，当对方挂断电话的时候, SIM800 模 块会返回"NO CARRIER",并结束通话,输入 `ATH` 挂断电话

 ## Some Information Handy
 * 中国移动主要使用的GSM频段P-GSM-900: 935–954MHz（下行），GSM辅助频段DCS: 1805–1820MHz（下行）
 * 中国联通主要使用的GSM频段P-GSM-900: 954–960MHz（下行），GSM辅助频段DCS: 1840–1850MHz（下行）
 * ARFCN与频率转换工具: https://www.cellmapper.net/arfcn
 

 ## 参考资料
 * Crazy Danish Hacker的GSM Sniffing & Hacking[YouTube](https://www.youtube.com/playlist?list=PLRovDyowOn5F_TFotx0n8A79ToZYD2lOv)视频合集
 * 雪碧 0xRoot的网站 https://cn0xroot.com
 * 中国移动维基百科[中国移动](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E7%A7%BB%E5%8A%A8)
 * 中国联通维基百科[中国联通](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E8%81%94%E9%80%9A)