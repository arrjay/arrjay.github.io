---
date: 2014-04-07T00:53:39-04:00
title: "IRIX 6.2 Installation and patching"
categories:
  - "irix"
  - "crazyoldthings"
tags:
  - "howto"
---
- do the install, reboot
  - `install default` is the magic for inst selection.
- DHCP will give you back an address, but you need to configure DNS. routed(!) will hand you a default route.

```
echo 'nameserver 10.100.252.2' > /etc/resolv.conf
echo 'domain j.hack' >> /etc/resolv.conf
echo 'hostname' > /etc/sys_id
cat /etc/hosts|sed s/IRIS/xxxHOSTNAMExxx/g > /etc/hosts.1
mv /etc/hosts.1 /etc/hosts
```

- you can verify with `ping` - there is no `host` or `getent` here!

- after you do that, `inst` will be able to run off the network again :)

```
install compiler_eoe.sw64.lib compiler_dev.sw64.lib compiler_dev.sw.abi compiler_dev.sw64.lib dev.man.irix_lib nfs.sw.dskless_client eoe.sw.kdebug
install patchSG0002420 patchSG0003163 patchSG0003400 patchSG0003723
conflicts 1b 2b 3b 4b 5b 6b 7b 8b 9b 11b
conflicts 2b
go
```

- reboot machine *twice* `shutdown -i6 -g0 -y`

```
install patchSG0003911 patchSG0001645 patchSG0002000
conflicts
go
```

- then reboot machine

```
install patchSG0003704.eoe_hdr.lib patchSG0003704.eoe_sw.perf patchSG0003704.eoe_sw.unix
conflicts
go
```

- then reboot machine
