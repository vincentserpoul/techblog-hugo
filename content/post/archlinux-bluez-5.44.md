+++
date = "2017-04-03T16:22:00+08:00"
title = "Bluetooth activation at startup on arch linux"
description = "enabling bluetooth on bluez 5.44"
tags = [ "archlinux", "bluetooth" ]
topics = [ "archlinux", "bluetooth" ]
slug = "archlinux-bluetooth"
+++

## Why?

You might not have reach that point yet but bluez has been deprecating hciconfig and other tools.
In bluez 5.44, [it's not there anymore](https://bugs.archlinux.org/task/53110).

## What is the problem?

All hciconfig udev rules to activate bluetooth at startup won't work anymore.
Most forum posts solving bluetooth issues are now outdated.
Once I updated to bluez 5.44, my service leveraging hciconfig to activate bluetooth at startup didn't work anymore!

## What is the solution?

It's now actually documented in the [archwiki](https://wiki.archlinux.org/index.php/bluetooth)

```
Alternatively and instead of the custom service and udev rule, you can simply use the new AutoEnable feature introduced in BlueZ 5.35 by uncommenting  [Policy] and AutoEnable=true lines in /etc/bluetooth/main.conf.
```
Just uncomment and go, you don't need the service to activate bluetooth anymore, enjoy!