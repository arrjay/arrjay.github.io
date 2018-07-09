---
title: "customizing the linux console"
description: "linux console resolution, color, font options"
date: 2018-07-08T15:09:08-04:00
categories:
  - "linux"
  - "makeityours"
tags:
  - "howto"
  - "screenshots"
  - "fonts"
  - "colors"
ogimage: "/screenshots/f28-fantasque-sans-8x14.png"
---

# Customizing the linux console

If you have a little linux server, you don't need to go the full X11 route just to have a better time
when you need to directly administer it. You can directly customize the console!

## Fonts

The console here doesn't have niceties like an font renderer, and that's
[arguably a good thing](https://googleprojectzero.blogspot.com/2015/08/one-font-vulnerability-to-rule-them-all_13.html)
- so we use _bitmap_ fonts - the PSF format, described [here](https://wiki.osdev.org/PC_Screen_Font).
Searching for "psf fonts" returns slighty more useful results than "linux console font".

### Installing other fonts

Fonts are loaded into the linux console using the `setfont` utility. We actually have a trick up our sleeve
here, this utility can also load NetBSD and FreeBSD console fonts, so we have all of those at our disposal
as well.

While playing with fonts, I _definately_ recommand a VM for this - loading fonts with incomplete or corrupted
symbols is trivial, and getting back out can be tricky. Doing this work in a VM means all you need to do is
reset to get back to a sane state.

To load a font, we use `setfont /path/to/file` and it will attempt loading whatever you handed it as a font.
Unfortunately, the specifics of font support are up to your console driver past that. Notably, the classic
VGA console is very limited, but most systems are using a framebuffer console now.

<a name="fontdir"></a>Unfortunately, distributions have not standardized on a console font path - Fedora/CentOS use
`/lib/kbd/consolefonts`, Debian/Ubuntu uses `/usr/share/consolefonts`.

Some distributions do include additional fonts as installable packages - the
[Terminus](http://terminus-font.sourceforge.net/) console font packages are available in Fedora/EPEL as
`terminus-fonts-console`, for instance.

While definately dated, [this page](http://v3.sk/~lkundrak/fonts/) from 2007 also goes over a wealth of
console fonts with downloadable links.

After installing a font, you may want to use the `showconsolefont` utility to render a collection of characters
from your font and see what they all look like.

### Converting fonts

<a name="ttf2psf"></a>If you have a TTF/OTF font and you'd like to try converting it for use in the console
take a look at
[ttf-console-fonts](http://urchlay.naptime.net/repos/ttf-console-fonts/about/) as a convenient wrapper around
all the utilities needed to render a modern font to bitmap and build a console font.

If you have an old school .FON file - that's _already a bitmap font_ - using `fon2fnts` and `fnt2psf` directly
from [psftools](https://www.seasip.info/Unix/PSF/) is the way to go here.

### Creating fonts

If you want to edit or create a new psf font,
[fontopia](https://sites.google.com/site/mohammedisam2000/fontopia) is an (as appropriate) console-based
bitmap font editor that can create or update linux PSF files directly.

### Making it permanent

#### In systemd

Systemd sets up the console font using `/etc/vconsole.conf` - the `FONT=` variable is set to the base name
(that is, the font name without .psfu/.psfu.gz extension) of a font from your [console font path](#fontdir).

### Screenshots

Here is a Fedora 28 system using the kernel default font:
![fedora showconsolefont kernel default font](/screenshots/f28-defaultkernelfont.png "fedora showconsolefont kernel default font")

Here is that same system using `eurlatgr`, the default font Fedora puts in `/etc/vconsole.conf`:
![fedora showconsolefont eurlatgr](/screenshots/f28-eurlatgr.png "fedora showconsolefont eurlatgr")

Note that here, the system has more characters available to it.

There are also some 8x8 fonts available, if you desire density over readability:
![fedora showconsolefont drdos8x8](/screenshots/f28-drdos8x8.png "fedora showconsolefont drdos8x8")

Here's my current preferred font, Terminus at 14 points, normal - `ter-v14n` is the font name here:
![fedora showconsolefont ter-v14n](/screenshots/f28-ter-v14n.png "fedora showconsolefont ter-v14n")

To show off a converted font, here's "3270 Semi-Narrow", converted via [ttf-console-fonts](#ttf2psf)
![fedors showconsolefont 3270seminarrow8x15](/screenshots/f28-3270seminarrow-8x15.png "fedora showconsolefont
3270seminarrow8x15")

Not all conversions go well though! Here's a quick attempt I did at converting
[monofur](https://www.dafont.com/monofur.font) which is missing `-`:
![fedora showconsolefont monofur-regular-8x13](/screenshots/f28-monofur-regular-8x13.png "fedora showconsolefont
monofur-regular-8x13")

Playing with different sizes also produced this mess, which is why I really recommend a vm:
![fedora showconsolefont monofur-regular-9x14](/screenshots/f28-monofur-regular-9x14.png "fedora showconsolefont monofur-regular-9x14")

As an example of converting .FON files, here's [dina](https://www.dcmembers.com/jibsen/download/61/) in the
console:
![fedora showconsolefont dina-6](/screenshots/f28-dina6.png "fedora showconsolefont dina-6")

This is actually terifically readable, but is missing linedraw characters, which becomes apparent when you run
[mc](https://midnight-commander.org/):

![fedora mc dina-6](/screenshots/f28-dina6-mc.png "fedora mc dina-6")

Here we cheerily loaded up the [swiss-8x16](https://svnweb.freebsd.org/base/head/share/syscons/fonts/swiss-8x16.fnt?view=log) font from FreeBSD, after a `uudecode` to turn into binary:

![fedora showconsolefont swiss-8x16](/screenshots/f28-swiss-8x16.png "fedora showconsolefont swiss-8x16")

Okay, as a final shot, here's [fantasque-sans](https://github.com/belluzj/fantasque-sans) in the console:
![fedora showconsolefont fantasquesansmono-8x14](/screenshots/f28-fantasque-sans-8x14.png "fedora showconsolefont
fantasquesanemono-8x14")

## Resolution

Modern linux attempts to read your display resolution information via
[EDID](https://en.wikipedia.org/wiki/Extended_Display_Identification_Data)
and maximize the resolution provided, to the best of the video card and monitor's capabilities. However, there
are two cases where that breaks down: KVM/IP and VM consoles - there's not a useful concept of a monitor. In
those cases, we can override the console system and force a specific resolution.

### For modern setups - the video parameter

The modern
<a name="drm"></a>[Direct Rendering Manager](https://en.wikipedia.org/wiki/Direct_Rendering_Manager)
takes a `video=` parameter, which understands standard VESA modes, so overriding to a standard mode is
trivial - something like `video=640x480` will force that particular mode.

To find out what modes are supported, boot the kernel without a `video=` parameter and look at the contents of
`/sys/class/drm/*/modes`. There is one folder under `/sys/class/drm` for each video output you have.

### For old setups - the vga parameter

If you're running older kernel versions, you may not have [drm](#drm) available for your particular video card,
so you get to play with the `vga` parameter documented in
[svga.txt](https://www.kernel.org/doc/Documentation/svga.txt) and see what is available.

## Colors

Linux's console setup is a terminal in it's own right, so as appropriate, you can set/reset the console colors
for when a a "return to defaults" escape code is sent.

### Per-user

Use the `setterm` command with the `--foreground --store` or `--background --store` parameters to set the
default foreground and background colors. Colors available are an interger index, `0` through `7`. You can
verify the colors output on your console by running the <a name="setterm-loop"></a>following:

```
for i in $(seq 0 7) ; do setterm  --forground $i ; echo hi ; done
```

Here is a screenshot of the default colors (note 0 is black, so not visible here):

![fedora setterm foreground loop](/screenshots/f28-setterm-loop.png "fedora setterm foreground loop")

Note that `--store` is the important bit to set a "default" color.

### Default foreground/background

Most console drivers support the `vt.color` kernel parameter - this is a pair of numbers specifying the
background, then foreground colors using the color set discussed above. For instance, I have rebooted my VM
with parameters here to have black on cyan - `3` is the background, `0` is the forground, giving `vt.color=0x30`
as the complete parameter:

![fedora vtcolor0x30](/screenshots/f28-vtcolor-0x30.png "fedora vt.color=0x30")

### The definition of color

One more trick up our sleve is to change the definitions of red, green and blue - the `setvtrgb` command can
do this at runtime, and the `vt.default_{red,grn,blu}=` parameters can establish the default parameters at
boot. Here's a screenshot of running `setvtrgb` and [our setterm loop](#setterm-loop):

![fedora setvtrgb example](/screenshots/f28-vtrgb-example.png "fedora setvtrgb example")

I got the [vtrgb](/reference/vtrgb) above from an attachment on
[this Ubuntu bug](https://bugs.launchpad.net/ubuntu/+source/kbd/+bug/730672)

## Console blanking

Finally, the default behaviour in the linux console is to blank or
[DPMS](https://en.wikipedia.org/wiki/VESA_Display_Power_Management_Signaling) off when possible after 10
minutes - that can be disabled, which is often useful for a server.

It's very simple to do, just add `consoleblank=0` as a kernel argument.
