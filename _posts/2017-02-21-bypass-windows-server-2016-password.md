---
layout: post
title: 物理接触绕过Windows Server 2016登录密码
---

## 有些标题党了，未开启BitLocker加密的Windows系统应该都可以使用此方法，修改用户的登录密码

前些日子装了个Windows Server 2016系统，好久没用，居然忘了登录密码。。。系统登录界面连重启选项都没有，只有网络控制和轻松使用，
连续**6**次密码错误，系统会假装第六次输入的密码正确，作出正在登录的假象，你欣喜地盯着正在登录提示下面的那个圈圈转啊转，转了好久，猛地弹出**密码错误**提示，
对于这种消耗手动暴力破解者时间以及耐心的做法，表示真心佩服。

### 原理

言归正传，该方法的基础是，目标Windows系统未开启BitLocker加密，使得拥有物理接触的我们可以替换系统 `C:\Windows\System32\Utilman.exe` 为 `C:\Windows\System32\cmd.exe` 
当然，名字保持不变，这样Windows系统每次调用轻松使用(Utilman.exe)，实际运行的二进制程序为cmd.exe。有了命令行窗口，可做的事情就很多了，何况是管理员级别的命令行窗口。

在支持NTFS(Windows系统盘的文件系统，XP当我没说)文件系统挂载的环境下，替换文件的方法有：
* 开机计入恢复模式，一般可以获得一个cmd窗口
* 进PE，Windows PE, Linux PE(Ubuntu这种安装盘也算)，甚至macOS PE，替换，注意备份真实的Utiman.exe文件，当然，前提是USB口可用
* 如果物理接触等级很高，可以拆下硬盘，挂在到其他电脑上，当然，我们的需求一般是重置密码，如果是为了读系统盘里的敏感文件，进PE，复制即可，无需这么麻烦

### 操作

本人使用Windows Server 2016的USB安装盘(电脑没法进入恢复模式，已做好重装系统的打算。。。)
1. 开机按F10进入启动选择界面，选择USB安装盘
2. 启动USB安装盘，在安装系统界面，选择修复计算机，修复模式选择命令行
3. 命令行下默认安装盘盘符为X，默认目录是 `X:\sources\` ，

进入Diskpart查看计算机上的硬盘
```
diskpart
```
查看计算机连接的硬盘(物理硬盘)，通常需要重置密码的系统，其系统盘状态是在线，并且已经分配了盘符
```
list disk
```
查看目标系统盘盘符，一般是C，没有盘符的也可以手动分配，总之要挂载目标系统盘
```
list volume
```
备份Utilman.exe
```
copy C:\Windows\System32\Utilman.exe C:\
```
用cmd.exe替换Utilman.exe，询问是否覆盖该文件，Yes
```
copy C:\Windows\System32\cmd.exe C:\Windows\System32\Utilman.exe
```
重启，进入系统登录界面，点击轻松使用，弹出了我们需要的命令行窗口

![命令行窗口](/images/cmd.png "命令行窗口")

重置用户密码
```
net user [用户名] [新密码]
```
使用新密码登录，成功！

## 参考资料
* [https://www.lifewire.com/step-by-step-guide-to-resetting-a-windows-7-password-2626309](https://www.lifewire.com/step-by-step-guide-to-resetting-a-windows-7-password-2626309)
* [http://www.digitalcitizen.life/command-prompt-advanced-disk-management-commands](http://www.digitalcitizen.life/command-prompt-advanced-disk-management-commands)
