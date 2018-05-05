---
layout: post
title: Install Kali NetHunter on OnePlus One
---

Device: OnePlus One 16GB

Base ROM: Cyanogen OS 13.1

Target ROM: Kali NetHunter Nightly

[Kali NetHunter](https://www.kali.org/kali-linux-nethunter/) brings the goodies of a full-fledged Penetration Testing Distribution to an Android phone, which makes an ideal penetration testing device. 
OnePlus One is a then flagship android phone with still keen community support. It is also the "preferred" device by the Kali NetHunter developers. 
![Kali NetHunter](https://www.kali.org/wp-content/uploads/2014/08/nexus-nethunter-devices-2.png)

To install Kali NetHunter on the OnePlus One, you should firstly have the factory ROM setup, which in my case is the Cyanogen OS 13.1, and a custom recovery installed, which is most likely the TWRP. For the Kali NetHunter image, as we have chosen the Cyanogen OS 13.1 as the base ROM, so the Lollipop image from [Offensive Security](https://www.offensive-security.com/kali-linux-nethunter-download/) may not work, so you will have to download the nightly images from [Kali NetHunter Nightly](https://build.nethunter.com/nightly/), feel free to download the latest build. 
> For a fresh install, you will need a nethunter-generic-[arch]-kalifs-\*.zip as well as a kernel-nethunter-[device]-[os]-\*.zip.
> The kernel should be flashed last.
> The update-nethunter-generic-[arch]-\*.zip files are for updating your installation or if you wish to download the Kali rootfs inside the NetHunter app instead.

For the OnePlus One, you will need nethunter-generic-armhf-kalifs-full-rolling-[build number].zip as the SnapDragon 801 SoC onboard has a 32 bit armhf architecture, not 64 bit like the SD 810 and later SoCs, and kernel-nethunter-oneplus1-marshmallow-[build number].zip.

To sum up, these are the files needed:
* Cyanogen OS 13 fastboot zip(the download page for Cyanogen OS shows 404 at the moment, you might have to find your own way)
* [TWRP for OnePlus One](https://twrp.me/devices/oneplusone.html)
* nethunter-generic-armhf-kalifs-full-rolling-[build number].zip and kernel-nethunter-oneplus1-marshmallow-[build number].zip

Next, let's start flashing!
1. Make a full backup(include the internal storage, cause every partion will be erased in this case), and power off the phone;
2. Hold the volume up and the power button simultaneously for about three seconds to boot into fastboot mode;
3. Extract the archive file(aka the ROM) and flash all the images with fastboot:
    ```
    fastboot flash modem NON-HLOS.bin
    fastboot flash sbl1 sbl1.mbn
    fastboot flash dbi sdi.mbn
    fastboot flash aboot emmc_appsboot.mbn
    fastboot flash rpm rpm.mbn
    fastboot flash tz tz.mbn
    fastboot flash LOGO logo.bin
    fastboot flash oppostanvbk static_nvbk.bin
    fastboot flash recovery recovery.img
    fastboot flash system system.img
    fastboot flash boot boot.img
    fastboot flash cache cache.img
    fastboot flash userdata userdata_64G.img (or userdata.img if you have a 16GB model)
    ```
    Notes: make sure fastboot works in your extracted path, which is usually not a problem for Linux users, but if you are on Windows, the quickest way is grabbing the stand alone binaries from [this page](https://developer.android.com/studio/releases/platform-tools.html), extract it and move all the images there, this way you will have a working fastboot environment without Android Studio installation or Environment Variable setup.
4. Boot into the newly installed Cyanogen OS and finish the setup wizard before jumping to the TWRP recovery as __required__ by the Kali NetHunter image;
5. Reboot into the TWRP recovery, first flash nethunter-generic-armhf-kalifs-full-rolling-[build number].zip then kernel-nethunter-oneplus1-marshmallow-[build number].zip
6. Reboot into system

That's it.
You may also want to follow the [Post Installation Setup](https://github.com/offensive-security/kali-nethunter/wiki#50-post-installation-setup).

## Reference
* [https://github.com/offensive-security/kali-nethunter/wiki](https://github.com/offensive-security/kali-nethunter/wiki)
* [https://forum.xda-developers.com/oneplus-one/general/guide-return-opo-to-100-stock-t2826541](https://forum.xda-developers.com/oneplus-one/general/guide-return-opo-to-100-stock-t2826541)
