---
layout: post
title: Debrick WD MyCloud 3TB without Opening Case in Linux
---
## About
* created: 2015/05/16
* edited: 2017/12/31

## Requirements
1. A router running OpenWrt;
2. Recovery software from [Fox_exe](http://community.wd.com/t5/user/viewprofilepage/user-id/280916), download it [here](https://docs.google.com/file/d/0B_6OlQ_H0PxVRndzakJhRXZ3OHM/edit).

## Steps
### First, setup a TFTP server with OpenWrt

You need to make sure that dnsmasq has been installed on your router,

then open up `/etc/config/dhcp` and under the 'dnsmasq' section add the following lines (or if these lines already exist, adjust the values to match). 
```
option enable_tftp '1'
option tftp_root '/var/tftp'
```

create a folder on your OpenWrt router where the recovery system for debricking WD MyCloud will be stored
```
mkdir /var/tftp
```

__ATTENTION!__ `/var` is a link to/tmp, which will be flushed after reboot. Then ~~it~~ the missing folder will prevent dnsmasq form starting up. 

__Make sure__ you uncomment these options above beforing rebooting, or change the `tftp_root` folder to somewhere else if your router have plenty of space left.

Then, upload two files `bootimage` and `start.sh` in the downloaded root folder to `/var/tftp`。
Restart dnsmasq
```
/etc/init.d/dnsmasq restart
```

to apply these changes.

### Second, bring your WD MyCloud back alive!
1. Make sure that nmap is installed.

    In your computer's terminal 
```
sudo nping --dest-mac <Your WD MyCloud's MAC> -c 999 --icmp --icmp-type echo --icmp-code 0 --data-string WD-ICMP-BEACON 255.255.255.255
```

2. Power on your WD MyCloud(Get its IP address in `/var/dhcp.leases` if you have enabled dhcp of your router)

3. `telnet <Your WD MyCloud's IP>` from your computer. 
    If everything went well, you should be able to see 
```
Trying 192.168.1.2...
Connected to 192.168.1.2.
Escape character is '^]'.

/ #
```     

4. Troubleshoot the system or install a new one.
    I prefer clean debian. You can get it from the first reference bellow.
    
## How it works?
Well, basically "WD-ICMP-BEACON" is a backdoor, 

WD MyCloud will try to boot from a TFTP server if it receives the specific packet `--data-string WD-ICMP-BEACON` during startup. 

## Furthermore
1. All you needs is a TFTP server, I choose OpenWrt  +dnsmasq for convenience;
2. You can actually use any TFTP image to do the trick, the `bootimage` used here has limited functions(I even have to use netcat to upload the system recover image);
3. As for debricking in Windows, just follow Fox_exe's README.txt in the Recovery software folder. 

## Reference
* [http://community.wd.com/t5/WD-My-Cloud/Clean-debian-and-OpenMediaVault-on-WDMyCloud/td-p/785505](http://community.wd.com/t5/WD-My-Cloud/Clean-debian-and-OpenMediaVault-on-WDMyCloud/td-p/785505)
* [https://stelfox.net/blog/2014/07/using-openwrts-dnsmasq-as-a-tftp-server/](https://stelfox.net/blog/2014/07/using-openwrts-dnsmasq-as-a-tftp-server/)
* [http://www.nasyun.com/thread-24024-1-1.html](http://www.nasyun.com/thread-24024-1-1.html)
