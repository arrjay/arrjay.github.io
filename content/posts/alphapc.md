---
title: "AlphaPC 164 LX"
date: 2014-04-28T01:34:17-04:00
categories:
  - "hardware"
  - "windowsnt"
  - "srm"
  - "crazyoldthings"
tags:
  - "howto"
---

# Firmware on the AlphaPC 164 LX
This machine can only hold alphabios *or* SRM in flash memory. This means you cannot flip from booting NT to booting OSS things without reflashing the machine.

## To flash from alphabios to SRM:
 1. In the backend, get 'lx164nt.rom' to actually be the SRM firmware image. If using my puppet mess, netboot::flag::alphapc_164lx_defaultfw: 'srm' should do what you want.
 1. Press F2 during boot to enter setup
 1. Choose "AlphaBIOS Upgrade..." from the menu. Wait for it to finish scanning the floppy.
     - It should say "SRM Console 5.8-1 000621.1129" as the new version if all went well. Press F10 to do the flash. Power cycle the system (yes, the power switch!) to actually reboot.

## To switch from PC Console to Serial port in SRM:
- at the '>>>' prompt

```
set console serial
init
```

`init` reboots the system

## To switch back
```
set console graphics
```
