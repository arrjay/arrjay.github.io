---
title: "The Foundation of Containerization - chroot"
date: 2018-07-09T16:56:30Z
draft: true
---
# The foundation of containerization - chroot

There is a lot of current popularity around the concept of linux containers, and here I wanted to detail one of the underpinnings - the chroot
syscall.

## Why

Linux containers are not some mythical black box - they are a collection of files, combined with startup mechanics to perform an isolated task.
The chroot system is very much the foundation of this by allowing a program (and any programs it starts) to have a different concept of a base
filesystem. This means you can take many linux programs and any dependencies they have, transport them to another directory, and be able to
run them as if it was a different linux system. Understanding how to create and maintain chroots is an excellent first step to working with
more complex docker images.

## What chroot is Not

The chroot call is _not_ a full virtualization solution - the system is still running the main kernel, just with a different filesystem view.
With more modern kernels, the ABI has largely stabilized, but it is good to be aware of cases where a program may expect specific kernel
features or configuration.

Relatedly, a chroot system is not an emulator - it presents the same processor type and capabilities as the host system.

## History

via (wikipedia)[https://en.wikipedia.org/wiki/Chroot] - the chroot call actually predates Linux, it has been used for some time to test new
system installations. Interest in it as a security apex dates to 1991 or so, and some programs have been specifically written to use it as
a component isolation mechanism.

## Creating a chroot

### Cloning the running system

### Using a bootstrap program

#### debootstrap

#### yum/dnf

## Playing in the chroot


