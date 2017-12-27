---
layout: post
title: QGA(QEMU Guest Agent) Add Custom Command
---

## Preface
Suppose you have `virtio serial port` setup, check with
```
virsh qemu-agent-command ${VMNAME} '{"execute":"guest-info"}'
```

## Setup
* Host: Ubuntu 16.04 Desktop x64
* Guest: Ubuntu 16.04 Server x64
* Host: QEMU/KVM apt install
* Guest: QEMU Guest Agent git source

### Get QEMU Guest Agent Source Code
QEMU Guest Agent is part of the QEMU code repository, so we need to get the whole QEMU code base first.
```
git clone git://git.qemu.org/qemu.git
cd qemu
git submodule init
git submodule update --recursive
./configure
```
Some dependency packages may be needed.

### Overview
To add a custom command like this
```
virsh qemu-agent-command ${VMNAME} '{"execute":"${custom-command}"}'
```
to the QEMU Guest Agent, only two files need to be modified.

They are `qemu/qga/qapi-schema.json` and `qemu/qga/commands-posix.c`.

Of cource, if your guest OS is Windows, edit `qemu/qga/commands-win32.c` instead.

### 1. qapi-schema.json
Add this
```
##
# @NetConfigResult:
##
{ 'struct': 'NetConfigResult',
  'data': { 'errcode': 'int', '*errmsg': 'str' } }

##
# @net-config:
#
# Returns: @NetConfigResult
##
{ 'command': 'net-config',
  'data': { 'mac': 'str', 'ip_address': 'str',
      'ip_address_type': 'str', 'netmask': 'str',
      '*gateway': 'str' },
  'returns': 'NetConfigResult' }
```

1. The first part `NetConfigResult` is a custom struct in which the result of the command `net-config` will be returned(output).
    
    In this case, `errcode` and `errmsg` will be returned.

    The `*` before `errmsg` means that `errmsg` is optional.
    
    We will come to that later in the `commands-posix.c` part.

2. `'command': 'net-config'` means the newly added command is `net-config`, 

    and `'returns': 'NetConfigResult'` means that 
    the result of its execution is in the form of `NetConfigResult`, the struct we added before. 
    
    The `data` part is the input for our `net-config` command, it can be used like this
    ```
    virsh qemu-agent-command ${VMNAME} '{"execute":"net-config", 
                                        "arguments":{"mac": "00:00:00:00:00:00", 
                                        "ip_address": "192.168.122.173", 
                                        "ip_address_type": "ipv4", 
                                        "netmask": "255.255.255.0", 
                                        "gateway": "192.168.122.1"}}'
    ```
    Similarly, the `*` before `gateway` means its optional.

### 2. commands-posix.c
Define a new function for `net-config`
```
 NetConfigResult *qmp_net_config(const char *mac, const char *ip_address,
                                        const char *ip_address_type, const char *netmask,
                                        bool has_gateway, const char *gateway, Error **errp)
```
Replace `net-config` in `qmp_net_config` with your command name.

__Pay attention to the underline `_`, it replaces the dash `-` in `net-config`.__

Because the input variable `gateway` is made optional, we have a new bool type parameter `has_gateway`.

The returned `NetConfigResult` can be initialized by
```
NetConfigResult *result = g_new0(NetConfigResult, 64);
result->has_errmsg = true;
```

if `result->has_errmsg = true`, `errmsg` will be included in NetConfigResult.

__When passing value to `result->errmsg`, use `g_strdup_printf()` instead of memcpy or direct assignment.__

### Debug
`cd` to the root directory of qemu

```
make qemu-ga
sudo make install qemu-ga
```

Then run
```
sudo qemu-ga -v
```
to debug.

### Conclusion
There you have it, a custom command for QEMU Guest Agent.

I will not dive into how I implemented the `qmp_net_config` fuction, which is for sure a tedious work.

## Reference
* [https://qemu.weilnetz.de/doc/qemu-ga-ref.html](https://qemu.weilnetz.de/doc/qemu-ga-ref.html)
* [http://wiki.stoney-cloud.org/wiki/Qemu_Guest_Agent_Integration](http://wiki.stoney-cloud.org/wiki/Qemu_Guest_Agent_Integration)
* [https://danielfresh.github.io/2016/04/20/qemu-ga-windows/](https://danielfresh.github.io/2016/04/20/qemu-ga-windows/)
* [https://www.cnblogs.com/biangbiang/p/3222458.html](https://www.cnblogs.com/biangbiang/p/3222458.html)
* [http://blog.nsfocus.net/easy-kvm-virtualization/](http://blog.nsfocus.net/easy-kvm-virtualization/)
* [http://www.voidcn.com/article/p-qokqicls-bdc.html](http://www.voidcn.com/article/p-qokqicls-bdc.html)
