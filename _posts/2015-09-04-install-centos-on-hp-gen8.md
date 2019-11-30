---
layout: post
title: Install Centos 7.0 on HP Microserver Gen8
comments: true
---
I've tested on both RHEL and CentOS 7.0 using hpvsa-1.2.10-120.rhel7u0.x86_64.dd.gz

download the driver
uncompress the .gz file
rename it from .dd to .iso
copy it to a fat formatted USB key
when you boot the installer add 'modprobe.blacklist=ahci inst.dd' on the boot line.
The above command is new for RHEL7.x, for 6.x it was just 'blacklist=ahci dd'.

When the installer starts you will be prompted for the device, the file, and the package, for the driver update. When finished you have to select continue. (as hpvsa loads it may drop text on the screen where anaconda is waiting for input so the system may appear hung. If you press enter you get the driver options line refreshed)

In my case I think it was 1, 1, 1, c

When you get into the installer, you will see the HP Logical Volume in the disk selection.

At first I thought I was seeing two disks from the installer, but that was just a first glance mistake. My usb device showed up as sda at 1.90 GB, and the disk volume showed up as sdb at 1.90 TB. Once I smacked myself upside the head and selected the right device the installation continued.


http://h20564.www2.hpe.com/hpsc/swd/public/detail?sp4ts.oid=5249572&swItemId=MTX_9200a10168684afbbb4efce88a&lang=en-us&cc=us&swEnvOid=4176
