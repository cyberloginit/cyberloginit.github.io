---
layout: post
title: 一加一代使用联通4G及高通固件详解
---

## 摘要
使用fastboot刷Oxygen OS的固件，解决一加一手机刷Cyanogen OS后无法使用联通3G、4G网络的问题。

## 环境
* Windows/Linux/macOS
* fastboot [此处下载](https://developer.android.com/studio/releases/platform-tools.html)

## 前言
心血来潮买了张联通卡，满怀欣喜插入一加一（Cyanogen OS cm-13.1-ZNH2KAS1KN），网络一直是2G EDGE。想到因LTE牌照问题，一加一在国内不正式支持联通3G、4G。
官网说
> WCDMA和 FDD-LTE两种网络制式需国际漫游状态时才能使用

但面向海外的Cyanogen OS居然也不支持，着实让人惊讶。

先用中文搜索了一下这个问题，中文互联网社区有几个帖子试图解决这个问题，本方案的灵感也部分来源于此帖。

~~但此类帖子多流于教程，而非技术。本着看教程不如学原理的原则，我们需要从这些神秘兮兮的教程步骤里提取出隐藏的信息。~~

既然Oxygen OS支持联通3G、4G，说明该ROM的固件（其实是基带）是支持的。

## 过程

### 第一次尝试（失败）
本以为简单地刷个[Oxygen OS 2.1.4](https://forums.oneplus.net/threads/oxygenos-2-1-4-for-the-oneplus-one.425544/)的modem
```
fastboot flash modem NON-HLOS.bin
```
就能成功，结果开机后无法使用任何网络。

### 第二次尝试（失败
同样只刷modem，这次受上述帖子影响，使用了Oxygen OS 1.0的`NON-HLOS.bin`文件，刷完回到了Cyanogen OS 13，看了一下基带版本，居然是一样的。。。

### 第三次尝试（成功）
刷`cm-13.1-ZNH2KAS1KN-bacon-signed-fastboot-a03c8dbbd0.zip`时需要在fastboot模式分别刷从LOGO到system的各个文件（所谓的把系统恢复成完全原生的方法），
那个压缩包中有个flash-radio.sh脚本，可以“一键”刷所有固件
```
fastboot flash aboot emmc_appsboot.mbn
fastboot flash LOGO logo.bin
fastboot flash rpm rpm.mbn
fastboot flash modem NON-HLOS.bin
fastbootflash sbl1 sbl1.mbn
fastboot flash dbi sdi.mbn
fastboot flash oppostanvbk static_nvbk.bin
fastboot flash tz tz.mbn
```
~~不太理解其中的几个分区的功能，比如aboot，rpm，sbl1，dbi，oppostanvbk（暴露了一加的东家），但感觉这几个文件应该功能紧密相连（本文水平好像比自己鄙视的教程党高不到哪里去。。。），所以决定把这几个分区都刷了。~~~

还是使用Oxygen OS 2.1.4的固件。当然，有这个脚本方便多了，真是“一键”刷机。

重启后，忐忑不安的盯着开机界面，生怕刷成砖了，还好顺利进入系统，这时基带版本已经是`.4.0.1.c7-00013"`。

Cyanogen OS 13和Oxygen OS 1.0都是`DI.3.0.c6-00241`，看版本号，还是Oxygen OS 2.1.4的基带更新啊（难道CM和一加闹掰后Cyanogen OS也不给好好做了）。

如此看来，固件（Firmware）还是用Oxygen OS以及对应的氢OS的最新版比较好，即使这版Oxygen OS只是个社区版。

## 高通固件说明
### aboot emmc_appsboot.mbn
application processor(对应baseband processor) boot loader；

emmc(embedded MMC)，Android手机常用的存储设备，当然各个固件也存储其中；

mbn(Multi Boot Image)，使用[readmbn](https://github.com/openpst/readmbn)可查看文件内容

### LOGO logo.bin
没什么好说的，开机动画，其实是个压缩文件，网上有大量自定义教程。

### rpm rpm.mbn
Resource Power Manager，负责设备的电源管理等。

### modem NON-HLOS.bin
基带系统固件

NON-HLOS(NON High Level OS)即基带处理器的系统，与HLOS(High Level OS) Android对应；

里面包含GNSS(Global Navigation Satellite System)，如GPS，子系统的控制模块；

还有mDSP(modem Digital Sigal Processor)等子系统的控制模块；

网上有泄露的高通某些型号基带源码，可以[自己编译](https://syshwid.blogspot.com/2016/10/build-qualcomm-modem-msm8626.html);

参考[此文](https://zhiwei.li/text/2015/08/16/%E4%BB%8E%E5%88%B7%E6%9C%BA%E5%8C%85%E6%88%96%E8%80%85%E5%B7%B2%E7%BB%8Froot%E7%9A%84%E6%89%8B%E6%9C%BA%E4%B8%AD%E6%8F%90%E5%8F%96%E5%9F%BA%E5%B8%A6%E5%9B%BA%E4%BB%B6/)

>NON-HLOS.bin 实际就是一个FAT16格式的磁盘映像
>有两种方法取得
>1. 从刷机包中解压缩
>2. 从root的手机中获取
挂载镜像
```
su -
mkdir mountpoint
mount -o loop NON-HLOS.bin mountpoint
cp -r mountpoint modem
umount mountpoint
```
>现在在modem目录下就有初步提取的底层驱动的文件了。

使用[Revskills Pro](https://forum.xda-developers.com/general/general/revskills-pro-edition-v2-08-6-t3489697)可以把`.b00`格式的文件转换成elf文件。

>提取过程中，如果遇到空缺的文件，会报告错误。你可以创建一个大小为0的空文件，文件名就为那个丢失的文件。然后重试。

>最后你将得到一个类似于 modem.elf的文件，可以放入IDA Pro中分析。

IDA Pro不能正确解析这种架构（QDSP6）的二进制文件，可以使用[这个](https://github.com/gsmk/hexagon)模块，也可以参考[这里](https://github.com/programa-stic/hexag00n)

### sbl1 sbl1.mbn
Secondary Boot Loader；

高通SoC(System on Chip)启动过程中有很多boot loader，一加一使用的MSM 8974AC把sbl1, sbl2, sbl3合并到了一个文件sbl1中。

### dbi sdi.mbn
未知

### oppostanvbk static_nvbk.bin
网络参数设定

### tz tz.mbn
Trust Zone
>Arm TrustZone technology is a System on Chip (SoC) and CPU system-wide approach to security. 

>TrustZone is hardware-based security built into SoCs by semiconductor chip designers who want to provide secure end points and a device root of trust.

## 吐槽
上文用到的几个文件都是在一加的英文论坛找到的，固件几十兆而已，可以直接下载，无需注册，有些链接可能打不开，请自备梯子。
[链接](https://forums.oneplus.net/threads/list-oneplus-one-firmwares-modems.421783/) 

联通4G的室内信号在我这里的确比移动好很多，前者基本全满，后者最好也只是一半。

最后一定要吐槽一下国内论坛（至此您可以关掉这个页面，去享受联通4G的畅快了）
1. 帖子里共享文件似乎很喜欢用百度云，百度这公司的产品能信吗？？？
    结果是没几天后链接就失效了，然后帖子下面一帮人求文件。。。
    
    与此形成鲜明对比的是，国外（主要是英文）的类似论坛分享的文件很多过了几年还可以下载。
    
    文件无法正常下载的后果是，教程的可用度会打很大折扣。

2. 好多不重要，也不大的文件，下载必须要注册，还要扣虚拟货币（说你呢，一加中文论坛，都什么年代了也不知道上HTTPS，看看你的国际版对手，全站HTTPS，对自己同胞就可以这样“不讲究”吗）。

## 后记
第一版成文于2016年7月10日，文中吐槽相关的文字都是这版遗留的；

第二版于2017年12月31日完成，揭开了高通SoC中诸多固件的神秘面纱，希望能为这个方向的研究者提供参考。

## 参考
* [https://forums.oneplus.net/threads/list-oneplus-one-firmwares-modems.421783/](https://forums.oneplus.net/threads/list-oneplus-one-firmwares-modems.421783/)
* [https://security.tencent.com/index.php/blog/msg/38](https://security.tencent.com/index.php/blog/msg/38)
* [https://zhiwei.li/text/2015/08/16/%E4%BB%8E%E5%88%B7%E6%9C%BA%E5%8C%85%E6%88%96%E8%80%85%E5%B7%B2%E7%BB%8Froot%E7%9A%84%E6%89%8B%E6%9C%BA%E4%B8%AD%E6%8F%90%E5%8F%96%E5%9F%BA%E5%B8%A6%E5%9B%BA%E4%BB%B6/](https://zhiwei.li/text/2015/08/16/%E4%BB%8E%E5%88%B7%E6%9C%BA%E5%8C%85%E6%88%96%E8%80%85%E5%B7%B2%E7%BB%8Froot%E7%9A%84%E6%89%8B%E6%9C%BA%E4%B8%AD%E6%8F%90%E5%8F%96%E5%9F%BA%E5%B8%A6%E5%9B%BA%E4%BB%B6/)
* [https://www.arm.com/products/security-on-arm/trustzone](https://www.arm.com/products/security-on-arm/trustzone)
