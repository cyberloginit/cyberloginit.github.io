---
layout: post
title: Ubuntu GNOME 16.04 Install Fcitx Chinese IME
---

The basic environment is a English version Ubuntu GNOME 16.04 and has no other language or input source installed, it is the typical install for most of the English-speaking users.
I like keeping my system clean and prefer the software working out of the box.

To have a Chinese IME installed, just
```
sudo apt install fcitx fcitx-googlepinyin fcitx-module-cloudpinyin
```
fcitx is just a framework, so you still need another IME like fcitx-googlepinyin, fcitx-module-cloudpinyin is optional.

Use `im-config` to change the default IME to fcitx and you are all set.

Don't forget to enable googlepinyin inside fcitx, by right click the keyboard icon on the indicator bar down left of the corner, click configure,
click the "+" down left of the window, untick "Only Show Current Language"(the system language is English, so you won't be able to find googlepinyin option in "Current Language"), type google and googlepinyin will show up.

What' more, you can configure many other options there. At least we have a working Chinese IME here, and no other hacking is needed, it just works out of the box, well, almost.

## Reference
*https://blog.yueyu.io/p/250 