---
layout: post
title: 'Reference:  OpenOCD, GDB'
---

<link rel="stylesheet" href="https://2ab9pu2w8o9xpg6w26xnz04d-wpengine.netdna-ssl.com/wp-content/cache/autoptimize/css/autoptimize_f0ba07fcfbfb99eca6dd6305271724f8.css"/>
# OpenOCD Pre-reqs
* Host:  Laptop or Beaglebone, with openocd installed, with some USB programmer
* openocd.cfg files: repo
#### To start OpenOCD, cd to directory with openocd.cfg, run `openocd`

# GDB Commands   
* Start GDB, with an ELF file if you want.  `gdb example.elf`
* Should work on the Beaglebone native gdb, because it is ARM, difference is eabihf vs eabi
* `(gdb) target remote localhost:3333`
* `monitor reset` , `monitor halt` , `monitor reset halt` 
* `monitor` actually issues OpenOCD commands thru GDB.
#### To verify an ELF file:  
#### `monitor verify_image /full/path/to/my.elf`  
Actually the search path is the directory that OpenOCD started in.
#### To load an ELF file into flash memory:  
#### `monitor flash write_image erase /full/path/to/my.elf`  
#### Use the erase parameter, to erase before writing.  According to openocd manual, you should assume other flash sectors will be erased too.
* To start gdb connected to remote target:  `gdb -ex "target remote localhost:3333" ~/my.elf`
  
# Telnet Commands
  
#### Single Shot Load
```
v1=copy-paste-your-path-to-elf-here.elf
(echo flash write_image erase $(pwd)/$v1 ; sleep 1) | telnet localhost 4444
```
#### Load Image Automatically
```
#!/bin/bash
fetch_and_load()
{
    echo Starting OpenOCD Server
    openocd -f ~/openocdconfigs/stm32f4discovery/openocd.cfg &
    echo Fetching and Loading new firmware: $v1
    wget ffn.bootladder.com:9001/$v1

    (echo reset halt; sleep 1) | telnet localhost 4444
    sleep 2
    (echo flash write_image erase $(pwd)/$v1 ; sleep 1) | telnet localhost 4444
    sleep 2
    (echo reset; sleep 1) | telnet localhost 4444
    sleep 2
    (echo shutdown; sleep 1) | telnet localhost 4444

}

v1=$(curl ffn.bootladder.com:9001/target/ffnbeagle-woodland-stm32discovery 2>/dev/null)
ls $v1 >/dev/null || fetch_and_load
```
This appears to work, but not sure if it works every time.  Hence the sleeps.  
* OpenOCD server is started and stopped for each write_image.  
* The curl retrieves the contents of the URL, which is a filename
* If the local system has that file, don't do anything.
* If not, grab that file and load it.
  
Loop the above to achieve automatic firmware updates
```
while true; do sh fetch-and-load-ffnbeagle-woodland-stm32discovery.sh ; sleep 1; done;
```
  
#### The above works when I only have 1 STM32F4 discovery board.
#### For my Atmel project, I may have 2 USB debuggers attached to the same host.
#### At large scale, a host will have many ttyUSB serial ports for bootloading
  
# Hmm, --enable-cmsis-dap not supported in my OpenOCD installation.
Ahh nice, `sudo apt install libhidapi-dev` , `./configure --enable-cmsis-dap` .  Props to http://www.signal11.us/oss/
