+++
date = "2019-04-16T11:07:08+08:00"
title = "installing docker on alpine linux"
description = "installing docker"
tags = [ "linux", "alpine", "rpi", "rpi0", "docker"]
topics = [ "linux", "alpine", "rpi", "docker"]
slug = "alpine-linux-rpi0-docke"
+++

## Prerequisites

- One raspberry pi zero W
- One sd card

## Enable all cgroups

On you local linux, mount the partition containing alpine files.

In the cmdline.txt, add the following:

```bash
...  cgroup_enable=memory cgroup_enable=cpuset swapaccount=1
```

Docker needs all cgroups enabled, so this will do the trick

## On you rpi0

As simple as:

```bash
apk add docker
rc-update add docker boot
```

reboot and check that everything is ok:

```bash
docker run --rm hypriot/armhf-hello-world
```

You should see some text if everything is working
