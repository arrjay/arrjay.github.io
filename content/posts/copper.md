---
title: "Copper (Darwin 8.0.1)"
description: "Installing Darwin 8.0.1 as a VM"
date: "2014-07-10T23:31:42-04:00"
categories:
  - "esx"
  - "darwin"
  - "crazyoldthings"
tags:
  - "howto"
---

# Installing Darwin 8.0.1 as a VM

Copper is an installation of Darwin 8.0.1. Installing Darwin is...finicky.

Avoid installing to ufs, that seems to work very poorly.

### ESXi configuration
- Virtual Machine - version 8 Hardware
- Hard disk 1 should be attached to IDE bus (0:0)
- 768MB of RAM seems to be plenty.
- E1000 ethernet adapter works here as well.
- For Guest OS I selected Other (FreeBSD (32-bit))
- Once up and running, you probably want to tweak settings to always boot from CD first - simply disconnect from image when we're done.

### Boot pass 1 - drop to shell
- initialize the partition table manually here using fdisk - 'fdisk -i -a hfs /dev/rdisk0' is what I did
- type exit at this point and the system will attept to reboot (actually shuts down for me)

### Boot pass 2 - install
- select your drive (1) to install on to, choose to use existing partitions
- root partition should be /dev/disk0s1 at this point - format as hfs and do a clean install
- install should proceed at this point.
- at the end, you are asked to seet a root password and hostname for the system.
- select 2 to attempt reboot (shutdown in my case) - you can disconnect the cdrom at this point.

## Optional Steps
Enable ssh by running 'service ssh start'. This will persist across reboots.

To actually configure static networking for ethernet devices use ncutil. 3.1.3 won't run at all on an unpatched darwin, 3.0b1 will run, and just dump a lot of localization errors. '[0 ]$' is the ncutil prompt in the below session.

```
# ncutil
[0 ]$ create-location Standard
[14 Standard]$ ls
[...] (<- find the ethernet adaptor device here)
[14 Standard]$ cd 30
[30 Ethernet Adaptor (en0)]$ setprop IPv4 method Manual
[30 Ethernet Adaptor (en0)]$ setprop IPv4 ip-address 10.100.252.5
[30 Ethernet Adaptor (en0)]$ setprop IPv4 subnet-mask 255.255.255.0
[30 Ethernet Adaptor (en0)]$ setprop IPv4 router 10.100.252.1
[30 Ethernet Adaptor (en0)]$ setprop DNS name-server 10.100.252.2
[30 Ethernet Adaptor (en0)]$ cd /
[0 ]$ setprop . CURRENT-LOCATION Standard
[0 ]$ exit
# scutil --set HostName copper
```
(edit /etc/hostconfig, add HOSTNAME=copper)
`# shutdown -r now`

```
# scutil
> open
> get State:/Network/Global/IPv4
> d.show
(read PrimaryService GUID)
> d.add DomainName j.hack
> set State:/Network/Service/(GUID)/DNS
> quit
```

### NIS Configuration
- set NISDOMAIN=j.hack in /etc/hostconfig and reboot
