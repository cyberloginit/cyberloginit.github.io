---
layout: post
title: Utelnetd Interface Parameter Stack Buffer Overflow
---

# Utelnetd Interface Parameter Stack Buffer Overflow

## Utelnetd

> utelnetd is a small and efficient stand alone Telnet server daemon.

The source code can be downloaded from [Pengutronix](https://public.pengutronix.de/software/utelnetd/), and the latest version is `0.1.11`.

> 2008-08-11	Marc Kleine-Budde <mkl@pengutronix.de>
>
>		- Revision 0.1.11 released
>		- added CFLAGS to linking stage
>		 (thanks to Remy Bohmer <linux@bohmer.net>)

There is also a SourceForge [project](https://sourceforge.net/projects/utelnetd/), but the latest version is `0.1.9`.

At present, the only major Linux distro still has Utelnetd in its packages repository is [Gentoo](https://packages.gentoo.org/packages/net-misc/utelnetd).

## Buffer Overflow

The stack buffer overflow vulnerability affects Utelnetd version from `0.1.7` to the latest `0.1.11`.

> 2003-08-06	Robert Schwebel <r.schwebel@pengutronix.de>
>
>		- Revision 0.1.7 released
>		- changed Makefile and utelnetd.c to work with BSD
>		  (thanks to Sepherosa Ziehau <sepherosa@myrealbox.com>)

When the compatibility with BSD was introduced in `Revision 0.1.7`,

```
if (interface_name) {
    strncpy(interface.ifr_ifrn.ifrn_name, interface_name, IFNAMSIZ);
    (void)setsockopt(master_fd, SOL_SOCKET,
            SO_BINDTODEVICE, &interface, sizeof(interface));
}
```

was changed to 

```
if (interface_name) {

    strcpy(interface.ifr_name, interface_name);

    /* use ioctl() here as BSD does not have setsockopt() */
    if (ioctl(master_fd, SIOCGIFADDR, &interface) < 0) {
        printf("Please check the NIC you specified with -i option\n");
        perror("ioctl SIOCGFADDR");
        return 1;
    }

    sa.sin_addr = ((struct sockaddr_in *)(&interface.ifr_addr))->sin_addr;
} else 
    sa.sin_addr.s_addr = htonl(INADDR_ANY);
```

Especially, `strncpy(interface.ifr_ifrn.ifrn_name, interface_name, IFNAMSIZ);` was changed to `strcpy(interface.ifr_name, interface_name);`.

The user-controlled `interface` parameter(`interface_name` local string variable) is copied to the `interface.ifr_name` element from the `struct ifreq interface` local variable without having its length checked.

The `interface.ifr_name` has a fix length of __16__ bytes on Linux systems like Ubuntu 18.04 64 bit.

```
valgrind --leak-check=full ./utelnetd -p 8000 -l /bin/sh -i "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
```

![Stack Buffer Overflow PoC](/images/utelnetd_interface_parameter_stack_buffer_overflow.png)

## Exploit

On a multi-user system with root and non-root users, if the `utelnetd` executable file has the __setuid__ attribute, malicious user could overflow the `interface.ifr_name` buffer and overwrite the EIP/RIP register to achieve privilege escalation and effectively take control of the whole system as `root`.

## Reference

* [https://public.pengutronix.de/software/utelnetd/](https://public.pengutronix.de/software/utelnetd/)
* [https://packages.gentoo.org/packages/net-misc/utelnetd](https://packages.gentoo.org/packages/net-misc/utelnetd)
* [https://sourceforge.net/projects/utelnetd/](https://sourceforge.net/projects/utelnetd/)
