---
title: "The Foundation of Containerization - chroot"
date: 2018-07-09T16:56:30Z
tags:
  - howto
categories:
  - linux
  - nix
  - software
ogimage: "/reference/folders.png"
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

Chroot typically only switches the base filesystem layers - processes in a chroot can typically still see other processes on the system
(though not meaningfully interact with them), network connectivity is shared with the host.

Typical chroots do _not_ have the wiring together to use an init system in the chroot. Notably, systemd does not operate *inside* a chroot,
[this document](http://0pointer.de/blog/projects/changing-roots) lays out the ways for systemd to interact with chroots pretty well.

Lastly, `chroot` is not generally user-accessible, you will need to be `root` on your system to do most chroot switching or creating tasks.
If users are defined, they can be switched to _after_ the chroot switch itself.

## History

Via [wikipedia](https://en.wikipedia.org/wiki/Chroot) - the chroot call actually predates Linux, it has been used for some time to test new
system installations. Interest in it as a security apex dates to 1991 or so, and some programs have been specifically written to use it as
a component isolation mechanism. A notable example is [sshd](https://github.com/openssh/openssh-portable/blob/master/README.privsep).

## Creating a chroot

For programs that are not explicitly designed to run in a chroot, there is a need to bring enough of an environment for them to run in.
This can include devices, libraries, and runtime filesystems. Due to the flexibility of the system, there is no one-size-fits all, or
even standardized solution.

### Cloning the running system

If you have the disk space, a simple way to create a new chroot environment is clone your existing system. Using `tar` is a straightforward way
to accomplish this. I sever `--exclude` arguments here, the first, `/chroot` prevents a copy loop (since `/chroot` is our destination) and the
other two prevent some errors when trying to copy socket files, which this system is using in those locations.

I use the stat command in the host to get the [inode](https://en.wikipedia.org/wiki/Inode) or file number of the `/` and `/chroot` directories.
Next, I switch in to the chroot, and run `stat` there to get the inode for the chrooted `/`. Note that it is the same number as `/chroot` in the
host.

```
# df -h /
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/fedora-root   15G  2.0G   14G  13% /
# pwd
/
# stat -c %i /
96
# mkdir /chroot
# stat -c %i /chroot
25374620
# tar cpf - --exclude /chroot \
  --exclude /var/lib/sss/pipes --exclude /var/lib/gssproxy/default.sock \
  --one-file-system / | tar xpf - -C /chroot
tar: Removing leading `/' from member names
tar: Removing leading `/' from hard link targets
# chroot /chroot
# pwd
/
# stat -c %i /
25374620
```

### Using a bootstrap program

While cloning your existing system, then modifying that is a possible chroot strategy, a more popular one is to use a set of programs to
build a new system in the chroot using minimal components from the host. Then, once there is an initial bootstrap, to switch in to the
chroot and customize things further from there. This allows for building environments tailored to a single program, or trying out parts
of a new linux system before making it your main operating environment. This is especially handy if you are looking at major system library
changes, since the only stable ABI the chroot needs is that of the linux kernel.

#### debootstrap

Debian and derivatives have tooling specifically for this, called [debootstrap](https://wiki.debian.org/Debootstrap). This tool will pull
packages from the debian archives and extract them to your new chroot. After initialization, you can jump right in and start working with
apt and such, just like a normal system.

```
# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	Fedora
Description:	Fedora release 28 (Twenty Eight)
Release:	28
Codename:	TwentyEight
# mkdir /chroot
# debootstrap bionic /chroot
[...a lot of progress and informational messages later...]
# chroot /chroot
# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04 LTS
Release:	18.04
Codename:	bionic
# 
```

#### yum/dnf

For systems using `rpm` style package management, the situation is a little trickier - they use a binary package database, that is not
guaranteed portable across versions - _especially_ not backwards compatible. Let's detail the simple case - same major versions - here.

```
# rpm -qa | wc -l
665
# mkdir /chroot
# rpm --root /chroot --initdb
# rpm --root /chroot -i \
    https://mirrors.kernel.org/fedora/releases/28/Everything/x86_64/os/Packages/f/fedora-release-28-1.noarch.rpm \
    https://mirrors.kernel.org/fedora/releases/28/Everything/x86_64/os/Packages/f/fedora-gpg-keys-28-1.noarch.rpm \
    https://mirrors.kernel.org/fedora/releases/28/Everything/x86_64/os/Packages/f/fedora-repos-28-1.noarch.rpm
warning: /chroot/var/tmp/rpm-tmp.WKYcr3: Header V3 RSA/SHA256 Signature, key ID 9db62fb1: NOKEY
# dnf --installroot=/chroot install -y "@Minimal Install" dnf fedora-release fedora-release-notes fedora-gpg-keys
# chroot /chroot
# rpm -qa | wc -l
287
```

I'm not going to detail the cross-release details needed for `rpm` chroots here, but the short of it is you need to use the
`rpmdb_dump` and `rpmdb_load` utilities to get the rpm database into a transportable format.
