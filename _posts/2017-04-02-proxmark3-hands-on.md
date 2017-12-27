---
layout: post
title: Proxmark3 Hands on
---

清明节假期，翻出吃灰已久的 Proxmark3.

系统：Ubuntu 16.04 64 bit Desktop

套件：Proxmark3 Easy by ELECHOUSE, PM3Easy V3.0

卖家给了很全的中文文档，按照其中的快速上手指导，把板子组装起来。接下来是驱动安装，卖家的文档中只给出了 Windows 环境下的安装方法，
这种开源硬件怎能用专有的软件，运行在闭源系统上呢？

好吧，其实我是信不过买家给的软件的安全性。那就开启 hard 模式，自己折腾 Linux 系统下的驱动安装方法。

打开 [Google](https://www.google.com/)，输入 Proxmark3 Ubuntu, [第一条链接](https://github.com/Proxmark/proxmark3/wiki/Ubuntu-Linux)。

总结一下步骤如下：
1. 安装依赖包（没有严格验证某些包是否真的必须）
```
 sudo apt-get install p7zip git build-essential libreadline5 libreadline-dev gcc-arm-none-eabi libusb-0.1-4 libusb-dev libqt4-dev ncurses-dev perl pkg-config wget
```
2. 板子通过 USB 线连接电脑，运行 `dmesg`
```
[10416.461687] usb 2-1.2: new full-speed USB device number 12 using ehci_hcd
[10416.555093] usb 2-1.2: New USB device found, idVendor=2d2d, idProduct=504d
[10416.555105] usb 2-1.2: New USB device strings: Mfr=1, Product=0, SerialNumber=0
[10416.555111] usb 2-1.2: Manufacturer: proxmark.org
[10416.555814] cdc_acm 2-1.2:1.0: This device cannot do calls on its own. It is not a modem.
[10416.555871] cdc_acm 2-1.2:1.0: ttyACM0: USB ACM device
```
3. 因为是 CDC 设备，跳到驱动安装部分。因为我的电脑是 64 位，通过这个[链接](http://sourceforge.net/projects/devkitpro/files/devkitARM/previous/devkitARM_r41-x86_64-linux.tar.bz2/download)，
下载编译所需的工具链，32 位的链接在[这儿](http://sourceforge.net/projects/devkitpro/files/devkitARM/previous/devkitARM_r41-i686-linux.tar.bz2/download)，
解压，移动文件，添加到 PATH
```
tar jxvf devkitARM_r41-x86_64-linux.tar.bz2
sudo mkdir /opt/devkitpro/
sudo mv  devkitARM /opt/devkitpro/
export PATH=${PATH}:/opt/devkitpro/devkitARM/bin/
#echo 'PATH=${PATH}:/opt/devkitpro/devkitARM/bin/ ' >> ~/.bashrc
```
最后一部是把 PATH 的改动永久化，个人认为没有必要。

4. 下载 Proxmark3 的源码并编译
```
git clone https://github.com/Proxmark/proxmark3.git
cd proxmark3
make
```
5. 进入 `proxmark3/client/` 目录，运行
```
sudo ./proxmark3 /dev/ttyACM0
```
proxmark3 需要以管理员权限运行才能访问设备。

6. 自行开发喽
```
proxmark3> help
```

## Reference
* https://github.com/Proxmark/proxmark3/wiki/Ubuntu-Linux/
* http://www.proxmark.org/forum/viewtopic.php?id=1668