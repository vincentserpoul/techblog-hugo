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
echo -ne " cgroup_enable=memory cgroup_enable=cpuset swapaccount=1" >> /run/media/youruser/xxx/cmdline.txt
```

Docker needs all cgroups enabled, so this will do the trick

## On you rpi0

As simple as:

```bash
sudo apk add docker
sudo rc-update add docker boot
sudo rc-service docker start
```

To make sure your docker env doesn't go in memory (512MB RAM won't bring you far...), set it on mmcblk0p3 by editing /etc/docker/daemon.json:

```bash
{
        "exec-root": "/media/mmcblk0p3/var/run/docker",
        "data-root": "/media/mmcblk0p3/var/lib/docker",
        "no-new-privileges": false
}
```

and restart the service:

```bash
sudo rc-service docker restart
```

```bash
docker run --rm hypriot/armhf-hello-world
```

You should see some text if everything is working
