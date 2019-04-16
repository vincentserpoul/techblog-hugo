+++
date = "2019-04-11T21:22:13+08:00"
title = "installing alpine linux on rpi0"
description = "step by step"
tags = [ "linux", "alpine", "rpi", "rpi0"]
topics = [ "linux", "alpine", "rpi" ]
slug = "alpine-linux-rpi0"
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

You'll create 3 partitions:

- 256MiB partition for alpine itself (have to be of type 0x0c - W95 FAT32 (LBA))
- 1GiB partition for the cache and config files
- 13.5GiB partition for your var folder

```bash
sudo fdisk /dev/sda
n - p - 1 - +256M - t - 1 - c - a - w
n - p - 2 - +1G - w
n - p - 3 - - - w
```

Format you first partition to fat, and the 2 others to ext4

```bash
sudo mkfs.vfat -F 32 /dev/sda1
sudo mkfs.ext4 /dev/sda2
sudo mkfs.ext4 /dev/sda3
```

Check where it is mounted:

```bash
lsblk

NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    1  14,8G  0 disk
├─sda1        8:1    1   256M  0 part /run/media/youruser/24CB-FC98
├─sda2        8:2    1     1G  0 part /run/media/youruser/4bb8adf3-f1c9-4367-8e38-7c09bad775ee
└─sda3        8:3    1  13,5G  0 part /run/media/youruser/42b25298-f013-483c-845c-9408e330bb75
```

Extract your previously downloaded alpine linux to the root of this partition:

```bash
sudo tar -C /run/media/youruser/24CB-FC98 -xzf ~/Downloads/alpine-rpi-3.9.2-armhf.tar.gz
```

unmount the sdcard:

```bash
umount /run/media/toto/10F8-4DDB
```

## On your rpi0

Put back the sdcard into the rpi and boot it up (with keyboard and screen).

Mount your 2 other partitions:

```bash
mkdir /media/mmcblk0p2
mkdir /media/mmcblk0p3
mount /dev/mmcblk0p2 /media/mmcblk0p2
mount /dev/mmcblk0p3 /media/mmcblk0p3
```

and run the setup:

```bash
setup-alpine
```

And let yourself be guided, don't forget to set the config and cache partition mmcblk0p2 to be saved to.

As of now, it seems there is a [bug](https://bugs.alpinelinux.org/issues/8025) for the dhcp lease, just follow [this comment](https://bugs.alpinelinux.org/issues/8025#note-11)
and add wpa_supplicant at boot:

```bash
rc-update add wpa_supplicant boot
```

Add a user with no pass, add its home directory to the config partition and add it to the sudoers

```bash
addgroup -S maintenance && adduser maintenance -G maintenance
apk add sudo
visudo
"%maintenance ALL=(ALL) ALL"
chown root:root /etc/sudoers
lbu add /home/maintenance
```

Bonus: change the welcome message:

```bash
echo "Welcome home!" > /etc/motd
```

And don't forget the commit!

```bash
lbu commit
```

## Back on your local computer

### apkovl

Unzip the local backup tarball (host.apkovl.tar.gz):

```bash
mkdir -p ~/alpine-install/lb
tar -C ~/alpine-install/lb -xzf /sda2mountpoint/host.apkovl.tar.gz
```

### ssh connection

Add your ssh key to the authorized_keys of the maintenance user and allow ssh connection

```bash
ssh-keygen -t rsa -C "youremail"
mkdir -p ~/alpine-install/lb/home/maintenance/.ssh
cat ~/.ssh/id_rsa.pub > ~/alpine-install/lb/home/maintenance/.ssh/authorized_keys
chmod 700 ~/alpine-install/lb/home/maintenance/.ssh/authorized_keys
chmod 600 ~/alpine-install/lb/home/maintenance/.ssh/authorized_keys
```

Also, allow public key login (PubkeyAuthentication yes) and remove password auth (PasswordAuthentication no ChallengeResponseAuthentication no) in /etc/ssh/sshd_config.

Add as well your 3rd partition to fstab:

```bash
echo "/dev/mmcblk0p3 /media/mmcblk0p3 ext4 rw,relatime 0 0" >> ~/alpine-install/lb/etc/fstab
```

apk add e2fsprogs

### repositories

Uncomment community packages as well, it will most probably be useful later on:

```bash
vi /etc/apk/repositories
uncomment http://dl-cdn.alpinelinux.org/alpine/latest-stable/community
apk update
```

### More disk avail

Diskless alpine is great as the fs is readonly and it's the safest and cleanest wa to install it.
Problem: the RAM is not THAT big in a rpi0, hence, once you start playing with things that are bigger, it doesn't work.
Solution: use the overlayfs, it will allow you to deport some folders in the last, biggest partition, in a persistent manner.

For example, if you want to use /var/test, you can overlay /var/test:

```bash
echo "overlay /var/test overlay lowerdir=/var/test,upperdir=/media/mmcblk0p3/var/test 0 0" >> ~/alpine-install/lb/etc/fstab
```

Recompress everything:

```bash
rm ~/alpine-install/host.apkovl.tar.gz
tar -czvf ~/alpine-install/host.apkovl.tar.gz -C ~/alpine-install/lb .
rm -rf ~/alpine-install/lb
sudo cp ~/alpine-install/host.apkovl.tar.gz /sda2mountpoint/
umount /sda2mountpoint
```

## Connecting through ssh

Connect via ssh with the maintenance user:

```bash
ssh -vv -i ~/.ssh/id_rsa_home_iot maintenance@192.168.1.100
```

Tips if you have errors, on you rpi, just uncomment the two following lines in /etc/sshd_config, and you should see logs in /var/log/auth.log:

```code
    SyslogFacility AUTH
    LogLevel INFO
```

## Enjoy
