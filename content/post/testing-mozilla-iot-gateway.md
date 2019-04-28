+++
date = "2019-04-29T00:23:23+08:00"
title = "testing mozilla iot gateway"
description = "alpine, docker, iot, zwave and zigbee"
tags = [ "alpine", "docker", "iot", "zwave", "zigbee", "mozilla-iot-gateway" ]
topics = [ "alpine", "docker", "iot", "zwave", "zigbee", "mozilla-iot-gateway" ]
slug = "mozilla-iot-gateway"
+++

# Mozilla iot gateway

[Here](https://iot.mozilla.org/gateway/) is what I am going to setup.
I am obviously not going to rewrite one more time the tutorial and docs from mozilla.
I will simply describe the specific setup I used, and the little things I setup to make it work.

## Prerequisites

- Raspberry pi 3b
- Zigbee dongle, zwave dongle [list of compatible hardware](https://github.com/mozilla-iot/wiki/wiki/Supported-Hardware) - I used Digi XStick (ZB mesh version) and Aeotec Z-Stick (Gen5)
- Zigbee Smart bulb (i used a philips hue), zigbee motion detector (I used philips motion sensor)

## Linux

I used my favorite lean alpine linux, with the setup as I explained in [this post](https://vincentserpoul.github.io/post/alpine-linux-rpi0/) minus the complexity brought up by the rpi0.

## Docker

Again, following my [previous post](https://vincentserpoul.github.io/post/alpine-linux-rpi0-docker/), minus the rpi0 complexity.

## Launching the iot gateway with docker

First thing first, if you are using a zwave dongle, you need to make sure /dev/ttyACM0 is owned by the dialout group, otherwise, it won't be available.
Assuming your alpine hostname is "home-gateway":

```bash
ssh -i ~/.ssh/YOURPUBKEY maintenance@home-gateway
sudo chown root:dialout /dev/ttyACM0
```

Launching the container with the needed devices:

```bash
sudo docker run -d --restart always --name mozilla-iot-gateway --net host -v /media/mmcblk0p3/mozilla-iot-gateway:/home/node/.mozilla-iot mozillaiot/gateway:arm
```

A little explanation:

- restart always will simply restart if the container crashes, but also on startup
- --net host is unfortunately compulsory for some addons of the gateway
- mounting a volume saved on mmcblk0p3 allows to have persistent storage (remember alpine otherwise runs in ram)

The booting time is **very** long, so be patient.

## Reaching you gateway

You should be able to use an external URL to reach your RPi, XXX.mozilla-iot.org.
It is not a simple website, it's also a PWA, allowing you to set it up as an app on your mobile phone!

## Configuration

From there on, I had no issues, you an simply follow the [mozilla tutorial](https://iot.mozilla.org/docs/gateway-user-guide.html)

## Conclusion

Here I am: docker, minimal host alpine linux and iot devices.
Next step: k3s!
