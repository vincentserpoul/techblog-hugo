+++
date = "2019-04-11T21:22:13+08:00"
title = "installing alpine linux on rpi0"
description = "step by step"
tags = [ "linux", "alpine", "rpi", "rpi0"]
topics = [ "linux", "alpine", "rpi" ]
slug = "alpine-linux-rpi0"
draft = true
+++

## Prerequisites

- One raspberry pi zero W
- One sd card

## Download Alpine linux

We'll use the almost latest version: 3.9.2 ([3.9.3 would not work for some reason](https://bugs.alpinelinux.org/issues/10224))
The one to use for RPI is the raspberry pi (surprising :P) [armhf](http://dl-cdn.alpinelinux.org/alpine/v3.9/releases/armhf/alpine-rpi-3.9.3-armhf.tar.gz).

## On your local computer (assuming you re using linux)

Mount the sdcard (should be automated, if not, you probably know how to do that and you probably don't need that tutorial)

List your disks:

```bash
sudo fdisk -l
Disk /dev/sda: 14,8 GiB, 15836643328 bytes, 30930944 sectors
```

The disk you just inserted should be available in the list. It most likely should be called /dev/sda.

Create your partition:

```bash
sudo fdisk /dev/sda
n - p - w
```

Change the partition type to 0x0c - W95 FAT32 (LBA) and make it bootable:

```bash
sudo fdisk /dev/sda
t - c - a - w
```

Format you partition to fat:

```bash
sudo mkfs.vfat -F 32 /dev/sda1
```

Check where it is mounted:

```bash
lsblk
sda           8:0    1  14,8G  0 disk
└─sda1        8:1    1  14,8G  0 part /run/media/toto/10F8-4DDB
```

Extract your previously downloaded alpine linux to the root of this partition:

```bash
sudo tar -C /run/media/toto/10F8-4DDB -xzf ~/Downloads/alpine-rpi-3.9.2-armhf.tar.gz
```

unmount the sdcard:

```bash
umount /run/media/toto/10F8-4DDB
```

## On your rpi0

Put back the sdcard into the rpi and boot it up (with keyboard and screen).

```bash
setup-alpine
```

and let yourself be guided (choose dropbear it's a bit lighter), don't forget

```bash
lbu commit
```

This will save your config locally.

reboot

As of now, it seems there is a [bug](https://bugs.alpinelinux.org/issues/8025) for the dhcp lease, just follow [this comment](https://bugs.alpinelinux.org/issues/8025#note-11)

and...

```bash
lbu commit
reboot
```

## Update your alpine linux

```bash
apk update
apk upgrade
lbu commit
```

## Connecting through ssh

For this, you will need to create a ssh key on your local computer.

Connect via ssh with root:

```bash
ssh root@192.168.1.XXX
```

add a user and your ssh key to its auth keys

```bash
addgroup -S extuser && adduser vincent -G extuser
echo "your pub key" > /home/vincent/.ssh/authorized_keys
chmod 600 /home/vincent/.ssh/authorized_keys
apk add sudo
visudo
"vincent ALL=(ALL) ALL"
lbu commit
```
