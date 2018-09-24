---
title: "Bootstrapping secrets - how I did it"
date: 2018-09-23T00:30:07-04:00
draft: true
---

# Initializing multi-device / multi-user secret sharing

## Though experiment - What is GRAS? What is TCB?

GRAS - Generally Recognized As Safe. This is a categorization I use of software
and systems that can be used to generate your secrets - thast is, they are
unlikely to be tampered with for the purpose of leaking your secrets.

TCB - Trusted Computing Base. This is the stack of software and systems you
trust, in our case, for managing and handling secrets.

## Hardware

Not your main computer, easily offline-able

For my purposes, merely using a computer that is dedicated to the task of
secret management is sufficient - this provides a gap from your general system
usage. Specifically, I use a raspberry pi, but again, I mainly stress using a
_different_ computer than your main machine. With respect to networking, this
particular raspi uses a USB wifi stick for connectivity, I merely unplug it
when switching to secret management.

A Pair of USB sticks

I actually use two USB sticks for the offline keying node - one is unencrypted,
containing a copy of the public key rings and encrypted password stores, the
other is encrypted and configured to hold swap and working private key files.
This configuration lets me keep key material encrypted at rest with a minimum
of OS fiddling.

A printer

I strongly recommend (and am actually going to walk through) creating your GPG
keypair and then running it through paperkey/qrencode to _print_ an offline
copy. This has the advantage of not being as dependent on a computer should key
recovery be needed. If you choose to go this route, you should use a
non-networked printer, plugged directly into the key management computer, that
you can trust at the same level as the key - the bits would go through it!

Optional - webcam

As a complement to the printer, having a webcam for your offline computer makes
restoring the keys from paper much easier, as I generate backups using QR
codes.

Optional - yubikeys

I actually use several yubikeys as my device target, this allows me to switch
computers without having to reissue keys specifically to them, and alleviates
some concerns around storing private keys on mainline computers.

## Software

Linux

Remember my thoughts on TCB and GRAS above - it is far easier to start with a
minimal linux distribution that to pare down the equivalent OS X or Windows
installations. If you want to feel particularly paranoid, you can choose
Tails, but for my purposes a mainline distribution is fine. You will, however
want to periodically boot the machine to make sure the software stays up to
date. For my implementation specifically, I use raspbian - that keeps up to
date and works well on my specific hardware box.

GPG

I'm using GPG here as that's the encryption stack supported by
[pass](https://www.passwordstore.org/) which is my intended application
for password management. I'll detail how that gets used in another article,
this is just detailing the environment for key generation.

Paperkey

I have scripts that wrap around [paperkey](http://www.jabberwocky.com/software/paperkey/)
for truly offline backup of the private key material. This software strips a
private key down to just the essentials for archival to paper - it can even
produce output to merely be written down.

### Setting up raspbian for the offline key manager

Install raspbian, configure keyboard, wifi, hostname, user password. Reboot for
net config. Install any updates and reboot. Install apg, cryptsetup-bin,
git, imagemagick, libccid, paperkey, pcscd, pcsc-tools, qrencode, scdaemon,
tmux, ykneomgr, yubico-piv-tool, zbar-tools.

#### Using systemd+udev for creating the transient encryption configuration

The details of handling and activating a transient encryption disk are mostly
scripted in
[cryptscratch.sh](https://github.com/arrjay/misc-scripts/blob/master/linux/cryptscratch.sh).
The script merely needs a device path to manage. I handle that through the
combination of systemd and udev: udev to identify and tag a specific device,
which systemd can start a service when available to do the setup steps. I'll
detail the systemd template first, then the udev rule to bind it to the device.

`/etc/systemd/system/cryptscratch@.service`
```
[Unit]
Description=cryptscratch for %i

[Service]
Environment=DEVICE=/dev/%I
Environment=DOIT=YES
Type=oneshot
RemainAfterExit=yes
ExecStart=/use/local/libexec/cryptscratch.sh
```

`/etc/udev/rules.d/90-cryptscratch.rules`
```
KERNEL=="sd*", ACTION=="add", ENV{DEVTYPE}!="partition", ENV{ID_SERIAL}=="FROM_UDEV_SERIAL", TAG+="systemd", ENV{SYSTEMD_WANTS}="cryptscratch@%k.service"
```

This adds a unit template file that takes the base name of a device to set up
encryption with, and the udev file performs device activation, as detailed
[here](https://coreos.com/os/docs/latest/using-systemd-and-udev-rules.html) but
with one significant addition - since this activates a template, I can pass
back to systemd the name of the device udev got - `%k` in the udev rule.

If you are doing this for your own USB stick, the serial number to match should
be available through running `sudo udevadm info /dev/sdX` when your USB stick
is plugged in.

After all that setup, upon boot, the device will repartiton, create new
transient crypt volumes, and replace /tmp with a folder on the crypt volume.
By default it will also reconfigure swap to run from an encrypted partition.

The othwer thing cryptscratch will do for you is set up home directories for
any user under the `/mnt/volatile/home` directory by copying /etc/skel and
setting permissions. I changed `pi`'s home directory by running `vipw` and
editing the `/home/pi` to `/mnt/volatile/home/pi` directly, then rebooting.

#### Udev rules for yubikeys

I use the following rule, found in [this guide](https://gist.github.com/ageis/14adc308087859e199912b4c79c4aaa4)
to allow the `plugdev` group (which `pi` is already a member of) access to
yubikeys via pcsc and/or scdaemon. Saved it as
`/etc/udev/rules.d/70-yubikey.rules`:

```
ACTION=="add|change", SUBSYSTEM=="usb", ATTR{idVendor}=="1050", ATTR{idProduct}=="0010|0110|0111|0114|0116|0401|0403|0405|0407|0410", GROUP="plugdev"
```

### Scripts for key management and generation

After bringing your secondary computer up-to-date and installing dependencies,
I have some repos to clone for key generation and managment. To help keep repo
maintenance separate for key generation, I first add a user to the system,
and do all maintenance operations as that user.

```
sudo useradd srcmgmt
sudo cp -R /etc/skel /home/srcmgmt
sudo chown -R srcmgmt:srcmgmt /home/srcmgmt /usr/local/src
sudo chsh -s $(grep /bash /etc/shells) srcmgmt
sudo -Hi -u srcmgmt
cd /usr/local/src
```

From here, anything where I refer to "clone a setup repo", run under this
account.

#### Keymat

[KeyMat](https://github.com/arrjay/keymat) is a collection of scripts for
creating GPG keys and creating QR code books. I clone the setup repo and
then have a command to create the offline keybook.

```
cd
/usr/local/src/keymat/create-offline-keybooks.sh
```

This will create a series of pdf files with the keys ready for printing. The
`public` prefix files are GPG public keys, and their corresponding private key
is in `private`. The `-ds` and `-ss` suffixes describe single-sided or
double-sided printing. To add the generated keys immediately to your system,
run the following:

```
zbarimg public-ss.pdf |sed '/QR-Code:16:.*/d' |sort|sed 's/^QR-Code:[[:digit:]][[:digit:]]://'|tr -d '\n'|base64 -d|tee pubkey.gpg|gpg --import
zbarimg private-ss.pdf |sed '/QR-Code:16:.*/d' |sort|sed 's/^QR-Code:[[:digit:]][[:digit:]]://'|tr -d '\n'|base64 -d|paperkey --pubring pubkey.gpg|gpg --import
```
