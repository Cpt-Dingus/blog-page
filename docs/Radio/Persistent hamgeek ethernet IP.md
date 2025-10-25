# Extract the rootfs
---
layout: default
title: Persistent HamGeek Ethernet IP
nav_order: 2
parent: Radio
---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


# Preamble 

When using a HamGeek (Pluto) SDR directly with an Ethernet cable, you will quickly discover that the interface IP does not automatically configure, forcing you to have to SSH into the SDR using the debug USB port every time you restart it to configure it using `ifconfig`. This guide will help you automatically configure the Ethernet IP.

# Extracting the rootfs

The rootfs is inside a U-Boot wrapped initramfs, we remove the U-Boot header to get the gzip compressed initramfs

```bash
mkdir work && cd work
dd if=/path/to/uramdisk.image.gz of=initramfs.gz bs=64 skip=1
```

You can verify it extracted properly:
```bash
file initramfs.gz
```

It should say `gzip compressed data`.

You can now extract it:
```bash
gunzip -c ../initramfs.gz | cpio -idmv
```

# Adjusting the IP
You can cd into the rootfs directory, then edit the following file:


```diff
./etc/init.d/S40network:L27
-   ETH_IPADDR=`fw_printenv -n ipaddr_eth 2> /dev/null | tr -cd '[a-zA-Z0-9]._-'`
-   ETH_NETMASK=`fw_printenv -n netmask_eth 2> /dev/null || echo 255.255.255.0 | tr -cd '[a-zA-Z0-9]._-'`
-   ETH_GW=`fw_printenv -n gateway_eth 2> /dev/null | tr -cd '[a-zA-Z0-9]._-'`

+   ETH_IPADDR=`fw_printenv -n ipaddr_eth 2> /dev/null || echo 192.168.1.10 | tr -cd '[a-zA-Z0-9]._-'`
+   ETH_NETMASK=`fw_printenv -n netmask_eth 2> /dev/null || echo 255.255.255.0 | tr -cd '[a-zA-Z0-9]._-'`
+   ETH_GW=`fw_printenv -n gateway_eth 2> /dev/null || echo 192.168.1.1 | tr -cd '[a-zA-Z0-9]._-'`
```

# Repack rootfs

Now that we changed the network config, we can repack the rootfs. Run this from inside the `rootfs` directory:

```bash
find . | cpio -o -H newc | gzip > ../initramfs.gz
cd ..
```

Now that the initramfs is packed, we can pack add the U-Boot header

```bash
mkimage -A arm -O linux -T ramdisk -C gzip -n "PlutoSDR ramdisk" -d initramfs.gz ./uramdisk.image.gz
```

The new image is now in the current directory as `uramdisk.image.gz`. You can copy it over to the SD card, replacing the old one

You're done! Upon restart, the SDR should automatically configure with the IP 192.168.1.10 and 192.168.1.1 as the gateway.


> Note: When connecting directly to the SDR, you should configure your computer's Ethernet interface to use 192.168.1.99 for the address and 192.168.1.1 for the gateway, as per the official HamGeek guide.
