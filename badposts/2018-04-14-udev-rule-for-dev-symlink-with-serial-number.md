---
layout: post
title: udev rule for /dev/ symlink with serial number
---
I discovered, you can do the following:  
```
KERNEL=="ttyUSB[0-9]*", ATTRS{idProduct}=="6001", ATTRS{idVendor}=="0403", SYMLINK+="ttyUSB%s{serial}"
```
The interesting part is the `%s{serial}`.  
%s is short for ATTRS{} , so the value of the ATTR is inserted there.  
`serial` is the name of an attr, like `idProduct`.  
So, now I can mark my FTDI's with their serial numbers and do cool stuff.  
I discovered this from https://txlab.wordpress.com/2016/06/14/udev-rules-for-ttyusb-devices/ , and then looked at the udev manual
