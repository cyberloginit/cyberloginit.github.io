---
layout: post
title: Update ThinkPad BIOS with USB Drive
---


My ThinkPad T460 is a decent linux laptop, however, due to the notorious support service of Lenovo, it seems that I have to but update the BIOS in a Windows environment.

After some googling, here is how to update the bios for a thinkpad with just a usb drive.

1. Download the [Bootable CD iso file](https://support.lenovo.com/us/en/downloads/ds112123), `r06uj54d.iso` in my case, it is actually a iso file, you can see that from the link, but somehow showed as a exe file.

Do not use the latest `r06uj55d.iso`, for it is not supported by the geteltorito tool, and you will get this message:
```
This data image does not seem to be a bootable CD-image
```
Probably, the notorious Lenovo tries to kill this method with this update.

2. Get the geteltorito tool, you can either download the perl srcipt from [here](https://userpages.uni-koblenz.de/~krienke/ftp/noarch/geteltorito/), or simply install it from the ubuntu repo:
```
sudo apt-get install genisoimage
```

3. Convert the iso file to a img file:
```
geteltorito -o bios.img r06uj54d.iso
```

4. Dump the img file to a USB drive:
```
sudo dd if=bios.img of=/dev/sdb
```

5. Do not unplug the USB drive and reboot, when the Lenovo logo appears press F12, choose the USB drive to boot;

6. In the BIOS update "system", follow the instructions and after several reboots, you are good to go.

## Reference
[https://workaround.org/article/updating-the-bios-on-lenovo-laptops-from-linux-using-a-usb-flash-stick/](https://workaround.org/article/updating-the-bios-on-lenovo-laptops-from-linux-using-a-usb-flash-stick/)