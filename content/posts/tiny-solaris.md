---
date: 2014-04-07T00:54:11-04:00
title: "Installing Solaris 8 on a tiny machine (sparcstation LX)"
categories:
  - "hardware"
  - "solaris"
  - "crazyoldthings"
tags:
  - "howto"
---
If the installer drops to a `#` prompt complaining about `bpgetfile` - don't worry, just type `exit` - you're doing an interactive install anyway.

At some point the installer will look at the amount of RAM in the box (64MB baby!) and stop. So we need to configure a disk, and swap *now*, then continue the installation.

- run 'format' to see the disks attached to the machine - you probably want disk 0, type `0` at the prompt.
- if asked to label the disk, choose `yes`
- this will set up a partition table for you, which you probably don't want. I've got a 4GB disk in the LX, but for simplicity, I create only two partitions - `swap` and `/`.
  - edit partition `6` (to remove it) - tag `unassigned`, starting cyl `0`, size `0`
  - edit partition `1` - tag `swap`, starting cyl `0`, `128M` or so
  - print the partition table to find where partition `1` ends
  - edit partition `0` - tag `root`, starting cyl (partition 1 + 1), size...I actually just used 3.5G and called it good.
  - `quit` the label menu, then `label` the disk, finally `quit` the partitioner.
- now activate the swap with `swap -a /dev/dsk/c0t0d0s1`
- finally type `exit` to get back to the installer.
