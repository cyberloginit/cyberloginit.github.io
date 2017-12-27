---
layout: post
title: QQ for Android Database Decipher
---

Android版QQ的数据库不像微信的那样使用SQLCipher加密，文件存放在
/data/data/com.tencent.mobileqq/databases/[QQ号码].db, 使用SQLite Browser
可以直接打开，但很明显，各个tables中的数据并非明文。

以table Groups为例，其中的各个群组名称并非用户设定值，分析后发现密文是由明文（用户设定值）与设备IMEI值做位异或得到，
~~推测其他table的加密方式亦类似~~测试发现，Friends table也采用这种加密方法。

考虑到微信的数据库EnMicroMsg.db密钥是设备IMEI值连接uin值字符串的MD5值前七位，QQ如此操作也是可以预见的。