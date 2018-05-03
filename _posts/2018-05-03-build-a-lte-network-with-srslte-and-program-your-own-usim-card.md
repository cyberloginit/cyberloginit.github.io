---
layout: post
title: Build a LTE Network with srsLTE and Program Your Own USIM Card
---

# Build a LTE Network with srsLTE and Program Your Own USIM Card
![LTE](https://upload.wikimedia.org/wikipedia/commons/d/d7/Evolved_Packet_Core.svg)

## Requirements
* A Linux (preferably Ubuntu 16.04.4 desktop 64-bit) machine (i5 Dual-Core CPU will do, preferably with a USB 3.0 port)
* A Software-defined radio (SDR) board with Ettus Universal Hardware Driver (UHD) or the bladeRF driver support, like USRP B210
* A Windows machine to run the closed source USIM card programming software
* A [PC/SC](https://en.wikipedia.org/wiki/PC/SC) compatible SIM card reader, like the MCR3512
* A USIM card compatible with the programming software
* A LTE handset, like the Oneplus One

## Build
>srsLTE is a free and open-source LTE software suite developed by SRS (www.softwareradiosystems.com).

Of the three well-known open source LTE stack(openLTE, srsLTE and OpenAirInterface), I find srsLTE have both the code elegance of openLTE and the completeness of OpenAirInterface.

With srsLTE, we are able to run a LTE network(eNodeB, MME, HSS and S/P-GW) on a single pc.

On a freshly installed 64-bit Ubuntu 16.04.4 desktop machine, install the required libraries with:
```
sudo apt-get install cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev

```

Then install the UHD driver for our USRP B210:
```
sudo apt-get install libuhd-dev libuhd003 uhd-host
```

Next, download and build srsLTE in the home directory with a non-root user:
```
cd ~
git clone https://github.com/srsLTE/srsLTE.git
cd srsLTE
mkdir build
cd build
cmake ../
make
make test
sudo make install
```

## Configuration
Change back to the source directory.
```
cd ~/srsLTE
```

### srsEPC
```
cp srsepc/epc.conf.example srsepc/epc.conf
cp srsepc/user_db.csv.example srsepc/user_db.csv
```

### srsENB
```
cp srsenb/enb.conf.example srsenb/enb.conf
cp srsenb/rr.conf.example srsenb/rr.conf
cp srsenb/sib.conf.example srsenb/sib.conf
cp srsenb/drb.conf.example srsenb/drb.conf
```

## Run
```
cd ~/srsLTE/srsepc
sudo srsepc epc.conf
```

Open another terminal:
```
cd ~/srsLTE/srsenb
sudo srsenb enb.conf
```

You can also tweak the epc.conf or the enb.conf file, like change the MCC and the MNC, or the DNS address.

__Note:__ The UE database file(user_db.csv) has to be within the same directory where you would run the srsepc command.

Search 'Available networks' from your phone, and you should seen the custom built 4G network among local networks.

![Networks](/images/ue_search_network.png)

## USIM Card
USIM card is a complex closed system. 

Usually, you will not be able to program a blank USIM card bought online with a open source software like [pySim](https://github.com/osmocom/pysim), 
because different manufacturers have different implementations for the USIM standard.


If you do not have USIM cards supported by pySIM, like [sysmoUSIM-SJS1](http://shop.sysmocom.de/products/sysmousim-sjs1), GrcardSIM, GrcardSIM2 and MagicSIM, 
a closed source software is needed, and most of the time, it is based on Windows.

Another way is to simply ask the seller to program the USIM cards for you. Both eBay and Taobao have plenty of them.

Just make sure the parameters: ICCID, IMSI, KI and OP/OPc are programmed accordingly.

### Save 150￥
I bought a USIM card from [Taobao](https://item.taobao.com/item.htm?id=564708442949) which costs me 30￥. The USIM card reader and the programming software combined costs up to 150￥.

![USIM](/images/taobao.jpg)

Note: If this link should fail, just search 'openlte' from Taobao.

So it seems that I will have to spend at least 180￥ for this.

Not so fast!

After digging through CSDN, a programmer website in China, I found a [interesting article](https://blog.csdn.net/u013429928/article/details/75319585). 

The USIM programming software used by the blogger is the same as the picture from the Taobao shop, although the later one is heavily watermarked.

![USIM programming software](/images/sim_personalize_tools.jpg) 

Both screenshots show that the USIM programming software is called `SIM Personalize tools(Copyright: GreenCard Co.,Ltd Ver 3.1.0)`. They are even of the same version!

The blogger has uploaded both a configuration file and a programing software to CSDN, but only gives the download link to the [parameters configuration file](https://download.csdn.net/download/u013429928/10137875). 

I have managed to find the download link for the programing software [here](https://download.csdn.net/download/u013429928/10125492).

Unfortunately, like most Chinese websites, CSDN requires credits to download. 

Fortunately, there are many sellers on Taobao offering CSDN downloading service which costs less than 3￥ for however many credits CSDN requires.

To save trouble, I have decided to offer it [here](https://mega.nz/#!KG5VCSxI!W8-nxXkc5cqQ8Bk1GFvinyNltKJuMZ7kgTIk-xCKaBs). You'd better have it scanned by some antivirus software for safety concern:)

As for the USIM reader, any PC/SC compatible smart card reader will do.

### Program the USIM Card
On Windows 10, just plug in the smart card reader and Windows will take care of installing the drivers needed.

Then, open SIM Personalize tools `GRSIMWrite.exe`, click `Read Card`, it should be able to read the parameters stored without error.

![Programming Card](/images/programming_card.png)

Next, write the card as you like.

For srsLTE, make sure the IMSI, KI and __OP__ values are identical with those in the `user_db.csv` file.

## Enbale IP Network
With the USIM card programmed above, you should be able to connect to LTE network.

To surf the Internet:
```
cd ~/srsLTE/srsepc
sudo su
./if_masq.sh [Interface for Internet connection]
```
You can get all the network interfaces of your machine with `ifconfig`, make sure you give `if_masq.sh` the name of the interface which connects to the Internet.

__Note:__ DO NOT run if_masq.sh with `sudo`, it will not work on Ubuntu.

srsEPC creates a virtual network interface `srs_spgw_sgi` with the IP address `172.16.0.0/24`. Your phone should be able to get an IP address like `172.16.0.2`.

![UE IP](/images/ue_ip.png)

The `if_masq.sh` script enables IP forward for our Ubuntu system and masquerades traffic from `srs_spgw_sgi` by the interface which connects to the Internet.

Next, add an Access Point Names(APN) in the Cellular network settings of your phone. 

![UE APN](/images/ue_apn.png)
![UE APN Setting](/images/ue_apn_setting.png)

Only two fields need to be filled, the first `name` can be anything you like; the second `APN` needs to be the same with settings in `epc.conf`, __srsapn__ by default.

Now, you can surf the Internet by the custom built LTE network, just like Wi-Fi!

The `Verizon` service provider name(SPN) value is set in the USIM card.

![UE Lock Screen](/images/ue_lock_screen.png)

Speed test with [FAST Speed Test](https://play.google.com/store/apps/details?id=com.netflix.Speedtest) from Netflix.

![Speed Test](/images/ue_lte_speed.png)

## Reference
* [https://github.com/srsLTE/srsLTE](https://github.com/srsLTE/srsLTE)
* [https://sourceforge.net/p/openlte/wiki/Programming%20you%20own%20USIM%20card/](https://sourceforge.net/p/openlte/wiki/Programming%20you%20own%20USIM%20card/)
* [https://blog.csdn.net/u013429928/article/details/75319585](https://blog.csdn.net/u013429928/article/details/75319585)
