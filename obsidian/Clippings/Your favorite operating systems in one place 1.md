---
title: "Your favorite operating systems in one place"
source: "https://netboot.xyz/"
author:
  - "[[netboot.xyz team]]"
published:
created: 2026-06-18
description: "netboot.xyz enables you to PXE boot many Operating System installers and utilities from a simple to use menu powered by the iPXE project."
tags:
  - "clippings netboot ipxe"
---
No install media

one bootloader, not a drawer of USB sticks

Fetched from upstream

pulls images over HTTP at boot time

x86\_64 · arm64

BIOS and UEFI builds

Self-hostable

run your own instance via Docker

Put the bootloader on a USB stick, an ISO, or your DHCP server. Power on, and pick what you want from the menu.

```
# write the image to a USB stick
dd if=netboot.xyz.img \
   of=/dev/sdX bs=1M
```

Network installers, live distros, and recovery utilities live in one menu — try an OS, install it, or repair a disk.

```
Linux Network Installs
Live CDs
Utilities
iPXE Shell
```

Built on iPXE's open-source firmware. Chainload it from GRUB, syslinux, or any UEFI shell.

```
chain --autofree \
  https://boot.netboot.xyz/menu.ipxe
```

## Quick start

1

Grab the image

Pick the format that matches how you boot — ISO for VMs and IPMI,.img for USB,.kpxe for TFTP.

`curl -O https://boot.netboot.xyz/ipxe/netboot.xyz.iso`

2

Boot from it

Plug in the stick, attach the ISO, or point your DHCP server at the bootloader file.

`dhcp-boot=netboot.xyz.kpxe`

3

Pick your OS

The menu loads. Navigate with the arrow keys, press enter, and it streams the kernel from the upstream mirror.

`▸ Ubuntu — Network Install`

## Get netboot.xyz

ISO, USB image, and PXE/TFTP and EFI builds for x86\_64 and arm64. Each release is published on GitHub with checksums.