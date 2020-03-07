---
layout: post
title: OpenOCD Target Practice
---
# How to actually write the flash?  I've never used GDB.
First start the OpenOCD server.  This is what I see:  `Info : at91samd20j18.cpu: hardware has 4 breakpoints, 2 watchpoints`
Then connect to the server with gdb.  Run the gdb command and then at the prompt type this:  `(gdb) target remote localhost:3333`  
I'm using arm-none-eabi-gdb.  If I just run native gdb, I get this output
```
Info : accepting 'gdb' connection from 3333
undefined debug reason 7 - target needs reset
Error: CMSIS-DAP: Write Error (0x04)
Error: CMSIS-DAP: Write Error (0x04)
Error: CMSIS-DAP: Write Error (0x04)
Error: CMSIS-DAP: Write Error (0x04)
```
But if I run arm-none-eabi-gdb, I get this
```
Info : accepting 'gdb' connection from 3333
undefined debug reason 7 - target needs reset
```
Now that I'm connected let's Read some memory addresses.  
```
(gdb) x 0x3FF00
0x3ff00:	0xdeadbeef
(gdb) x 0x3FE00
0x3fe00:	0x11456708
``` 
Great!  Seeing deadbeef is good; it had to come from the target's flash.  
Interesting, if I supply a bogus address it handles nicely.  This is the output of the OpenOCD server after doing a `(gdb) x 90000000`.
```
Error: CMSIS-DAP: Write Error (0x04)
Polling target at91samd20j18.cpu failed, GDB will be halted. Polling again in 100ms
Polling target at91samd20j18.cpu succeeded again
```
Let's try to write something.  Oh dang, doesn't work.
```
(gdb) x 0x3f100
0x3f100:	0xffffffff
(gdb) set {int}0x3f100 = 0xdeadbeef
Writing to flash memory forbidden in this context
```
Now let's go to the OpenOCD manual and look at their example.  It says to run gdb with an .elf file.  Let's do that.  Dang.  Same error.  
  
But, OpenOCD manual shows using the `load` command.  That worked!
```
(gdb) set {int}0x3F100=0xdeadbeef
Writing to flash memory forbidden in this context
(gdb) load
Loading section .text, size 0xfea0 lma 0x0
Loading section .debug_breadcrumbs, size 0x4 lma 0xfea0
Loading section .relocate, size 0x110 lma 0xfea4
Start address 0x74b8, load size 65460
Transfer rate: 1 KB/sec, 9351 bytes/write.
```
Well atleast I can write to RAM.
```
(gdb) set {int}0x20006000=0xdeadbeef
(gdb) x 0x20006000
0x20006000:	0xdeadbeef
```
OK, here's another lead.
```
(gdb) monitor flash probe 0
flash 'at91samd' found at 0x00000000
```
This shows that the flash bank 0 is found at address 0x0.  This is correct.  
Another thing I can do is erase_check
```
(gdb) monitor flash erase_check 0
successfully checked erase state
	#  0: 0x00000000 (0x4000 16kB) not erased
	#  1: 0x00004000 (0x4000 16kB) not erased
	#  2: 0x00008000 (0x4000 16kB) not erased
	#  3: 0x0000c000 (0x4000 16kB) not erased
	#  4: 0x00010000 (0x4000 16kB) erased
	#  5: 0x00014000 (0x4000 16kB) erased
	#  6: 0x00018000 (0x4000 16kB) erased
	#  7: 0x0001c000 (0x4000 16kB) erased
	#  8: 0x00020000 (0x4000 16kB) erased
	#  9: 0x00024000 (0x4000 16kB) erased
	# 10: 0x00028000 (0x4000 16kB) erased
	# 11: 0x0002c000 (0x4000 16kB) erased
	# 12: 0x00030000 (0x4000 16kB) erased
	# 13: 0x00034000 (0x4000 16kB) erased
	# 14: 0x00038000 (0x4000 16kB) erased
	# 15: 0x0003c000 (0x4000 16kB) not erased
```
While this is correct, I already know that the page size is 64 bytes and the erase row size is 4 pages, 256 bytes.  Definitely not 16kB.
  
I learned here, that when I say `(gdb) monitor flash erase_check 0` , what happens is, monitor tells gdb to pass `flash erase_check 0` to OpenOCD.  So flash erase_check is an OpenOCD command, documented in the manual.
We can do
* flash erase_sector num first last
* flash erase_address [pad] [unlock] address length
* flash fillw address word length 
* flash fillh address halfword length 
* flash fillb address byte length
* flash write_bank num filename [offset]
* program filename [verify] [reset] [exit] [offset]

# What happens with multiple debug interfaces on 1 host?



# SAMD20 Custom Board with Atmel-ICE
```
# Atmel-ICE JTAG/SWD in-circuit debugger.
interface cmsis-dap
cmsis_dap_vid_pid 0x03eb 0x2141

# Chip info
set CHIPNAME at91samd20j18
source [find target/at91samdXX.cfg]
```
This is the output from lsusb -v:  
```
Bus 001 Device 002: ID 03eb:2141 Atmel Corp. 
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass          239 Miscellaneous Device
  bDeviceSubClass         2 ?
  bDeviceProtocol         1 Interface Association
  bMaxPacketSize0        64
  idVendor           0x03eb Atmel Corp.
  idProduct          0x2141 
  bcdDevice            1.01
  iManufacturer           1 Atmel Corp.
  iProduct                2 Atmel-ICE CMSIS-DAP
  iSerial                 3 J41800075898
```
Let's just see what happens if I change those IDs.  Interesting, it still works fine, even if I delete the whole line cmsis_dap_vid_pid 0x03eb 0x2141.  
Next with that whole line deleted, I put a spelling mistake,  
`interface cmsis-dapABC`
```
Open On-Chip Debugger 0.8.0 (2014-10-20-23:18)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.sourceforge.net/doc/doxygen/bugs.html
Error: The specified debug interface was not found (cmsis-dapz)
The following debug interfaces are available:
1: parport
2: dummy
3: ftdi
4: usb_blaster
5: amt_jtagaccel
6: ep93xx
7: at91rm9200
8: gw16012
9: usbprog
10: jlink
11: vsllink
12: rlink
13: ulink
14: arm-jtag-ew
15: buspirate
16: remote_bitbang
17: hla
18: osbdm
19: opendous
20: aice
21: bcm2835gpio
22: cmsis-dap
Runtime Error: incorrect_openocd.cfg:2: 
in procedure 'script' 
at file "embedded:startup.tcl", line 58
in procedure 'interface' called at file "incorrect_openocd.cfg", line 2
```
So here we see the "Interfaces" we can choose.  
Next I examined the Chip Info.  
Deleting the line  `set CHIPNAME at91samd20j18` had no change, even setting the name to a bogus value did not affect anything.  
So clearly the important lines are 
```
interface cmsis-dap
source [find target/at91samdXX.cfg]
```
So, what is inside target/at91samdXX.cfg ?  
```
#
# samdXX devices only support SWD transports.
#
source [find target/swj-dp.tcl]
```
OK, what is SWJ-DP ?  What is SWJ?  Let's see the file.  
```
# ARM Debug Interface V5 (ADI_V5) utility
# ... Mostly for SWJ-DP (not SW-DP or JTAG-DP, since
# SW-DP and JTAG-DP targets don't need to switch based
# on which transport is active.
...
...
if [catch {transport select}] {
  echo "Error: unable to select a session transport. Can't continue."
  shutdown
}

proc swj_newdap {chip tag args} {
 if [using_hla] {
     eval hla newtap $chip $tag $args
 } elseif [using_jtag] {
     eval jtag newtap $chip $tag $args
 } elseif [using_swd] {
     eval swd newdap $chip $tag $args
 }
}
```
The comment at the top explains, SWJ is for debug interfaces that support both JTAG and SWD.  My Atmel-ICE is one of those.  
proc swj_newdap is just a switch to select the command, in my case it would be using_swd so `eval swd newdap $chip $tag $args`.  
Hmm where is `using_swd` defined, and what is `swd newdap` ?  
Back to at91samdXX.cfg , we see this line:
```
swj_newdap $_CHIPNAME cpu -irlen 4 -expected-id $_CPUTAPID

set _TARGETNAME $_CHIPNAME.cpu
target create $_TARGETNAME cortex_m -endian $_ENDIAN -chain-position $_TARGETNAME

$_TARGETNAME configure -work-area-phys 0x20000000 -work-area-size $_WORKAREASIZE -work-area-backup 0
```
swj_newdap takes 3 args, chip tag args.  What's a tag?  Whatever.  
Then there's `target create $_TARGETNAME ...`  
We're configuring a cortex_m CPU with endianness.  
Then configuring it with a RAM work area?  don't know what that's for... does it clobber app ram?
  
Then at the end there's this:
```
set _FLASHNAME $_CHIPNAME.flash
flash bank $_FLASHNAME at91samd 0x00000000 0 1 1 $_TARGETNAME
```
I guess we are supplying this information to GDB?



# SAMD20 Custom Board with SAMD20 XPlained Pro
# SAMD20 XPlained Pro Self
# SAMD10 XPLained
# SAMD21 XPLained
# Arduino Leo

# How does the Raspberry Pi GPIO Driver Interface work?
It's the bcm2835gpio Debug Interface.
It says See interface/raspberrypi-native.cfg for a sample config and pinout.  
Let's take a look.
```
interface bcm2835gpio

# Transition delay calculation: SPEED_COEFF/khz - SPEED_OFFSET
# These depend on system clock, calibrated for stock 700MHz
# bcm2835gpio_speed SPEED_COEFF SPEED_OFFSET
bcm2835gpio_speed_coeffs 113714 28

# Each of the JTAG lines need a gpio number set: tck tms tdi tdo
# Header pin numbers: 23 22 19 21
bcm2835gpio_jtag_nums 11 25 10 9
```
Interesting, there's another one.
```

interface sysfsgpio

# Each of the JTAG lines need a gpio number set: tck tms tdi tdo
# Header pin numbers: 23 22 19 21
sysfsgpio_jtag_nums 11 25 10 9

# At least one of srst or trst needs to be specified
# Header pin numbers: TRST - 26, SRST - 18
sysfsgpio_trst_num 7
# sysfsgpio_srst_num 24
```
Notice the pin numbers for JTAG are the same.  
OK, bcm2835 is more directly accessing the hardware.  
But sysfsgpio sounds very interesting, like maybe I can port it to the Beaglebone?


# References, Links
http://thehackerworkshop.com/?tag=openocd
