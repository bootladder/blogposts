---
layout: post
title: Debugging a Bootloader with GDB and OpenOCD
---
I have a working build of a bootloader.  Let's flash it.  I have a binary and an ELF.  First let's flash the binary.  
I'm using a SAMD20 custom board with an Atmel-ICE driven by a Beaglebone running OpenOCD, and I'm driving the Beaglebone with SSH.
Let's configure the Beaglebone's OpenOCD conf.

**Hmm, arm-none-eabi gcc and gdb are 700mb disk.  No thanks?**  
Let's see what we can do with telnet.
```
telnet localhost 4444
```
Cool, halt and reset work.  
To read some memory use these
```
> mdw 0x3ff00
0x0003ff00: 2110002a 
```
Let's load an image
```
> load_image /tmp/binarg1
8180 bytes written at address 0x00000000
downloaded 8180 bytes in 0.327776s (24.371 KiB/s)
> reset  
```
Looks like it worked.  Interestingly there is no LED toggle on the debugger.  I guess edbg puts it in but OpenOCD doesn't.  

Let's look at the vector table.
```
> dump_image /tmp/bindump1 0x0 0x100
dumped 256 bytes in 0.011345s (22.036 KiB/s)
-------
-------
debian@FFNBeagle-A-0:/tmp$ od -tx1 -w4 -Ax bindump1 -v
000000 00 80 00 20
000004 25 07 00 00
000008 65 07 00 00
00000c 0d 07 00 00
000010 00 00 00 00
000014 00 00 00 00
000018 00 00 00 00
00001c 00 00 00 00
000020 00 00 00 00
000024 00 00 00 00
000028 00 00 00 00
00002c 65 07 00 00
000030 00 00 00 00
000034 00 00 00 00
000038 65 07 00 00
00003c 65 07 00 00
000040 65 07 00 00
000044 65 07 00 00
000048 65 07 00 00
00004c f1 06 00 00
000050 fd 05 00 00
```
  
# Weird problem, what is going on?
As I said, I have a working bootloader and a working application that can be bootloaded and run.  I used edbg, which always worked flawlessly when -p for programming, -e for erasing.  After the firmware is flashed, the application is loaded by serial port, ttyUSB.  
Very weird, I used OpenOCD to flash the bootloader firmware, it was OK, then I used ttyUSB to load the application, that was also OK, but then when I reset the target, it would jump to the application and hit some hard fault or something and reset.  WTF is OpenOCD doing differently?  I'm not even using OpenOCD for erasing anymore, I use edbg because I know for sure it erases, so it must be the programming.  I'm using binary images, not ELF's so it really should just dump the memory, I don't see how there could be a difference.  
Well, I know the next thing to try.  Try all the different commands for writing flash memory from a file.  I was using load_image blah.bin , where my blah.bin should start at address 0x0 so no more parameters are supplied to load_image
  
OK, before I do that, I should be able to find out what exactly went wrong by analyzing the flash memory contents on target.  
It looks like the bootloader is OK but then fails to jump to the application.  Let's check the pointers.  

The app loads from 0x2000, and puts a pointer to while(1) at 0x2100.
```
> mdw 0x2100
0x00002100: 00002229 
```
Check the listing for address 2229, that is in app space, check the app listing file.  Well this is weird, jenkins appears to be building a different binary from my laptop.  GCC version is different.  Crap.  We're going to have to make Jenkins a single point of truth here, and I'll have to publish the linker map file so I can view it on my laptop.  
  
Well not sure what's happening but things seem to be working on OpenOCD again.
