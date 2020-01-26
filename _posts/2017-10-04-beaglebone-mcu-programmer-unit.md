---
layout: post
title: beaglebone MCU programmer unit
---
  
I'm flashing Atmel ARM microcontrollers with a USB programmer dongle.  
  
```
https://github.com/bootladder/edbg
```
  
Got a fresh beaglebone running, grabbed the IP from the local router.  
In order to get the git clone to work I had to get the time right.  
```
apt-get install ntp
```
  
the compiled edbg binary is not included so it must be installed.  
Let's write the install script here.  
  
```
cp libudev.h /usr/include;
cp libudev.so /lib ;
make;
```
  
Hmm looks like the library I included in there is out of date.  libc perhaps?  
  
Fortunately it worked after a 
```
apt-get install libudev-dev 
```
  
So I'll just take that copy of libudev.so out of the repo.  
  
So now it looks like this
```
apt-get install libudev-dev
make;
cp edbg /bin/;
```
  
Then I run a 
```
root@beaglebone:/tmp# edbg -l
Attached debuggers:
  J41800059345 - Atmel Corp. Atmel-ICE CMSIS-DAP
```
  
Sweet!

