---
layout: post
title: STM32F769 Discovery Kit
---
  
I got a STM32F769 Discovery Kit to play with.  Let's bring it up, under control.  
   
The first experiment I need to do is making sure I can restore the flash to the factory state.
1. Read the flash and write it back
1. Compare the included demo project HEX file with the flash on board (should be the same)
1. Build the demo project from source and flash it
   
Once I do this I can safely make stupid mistakes :)
  
I'll do this the easy way first, and then switch to standalone OpenOCD later.  Go to <www.openstm32.org> and download the "System Workbench for STM32".  
This includes the Eclipse-based IDE, build toolchain and flash programming.  
  
While that downloads let's find the included HEX file for the demo firmware, and the IDE project file we'll use.  
```
steve@McBain:~/prog/embedded/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin $$$ f *.hex
./Binary/STM32CubeDemo_STM32769I-DISCO_V1.4.0.hex
./Binary/STM32769I-DISCO_DEMO_V1.0.0_FULL.hex

steve@McBain:~/prog/embedded/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin $$$ f *proj*
./SW4STM32/STM32F769I-Discovery_Demo/.cproject
./SW4STM32/STM32F769I-Discovery_Demo/.project
./MDK-ARM/Project.uvprojx
```
Isn't `find` nice?   
  
OK it downloaded so let's install it.
```
steve@McBain:~/Downloads $$$ sudo apt install gksu    # first install gksudo
...
steve@McBain:~/Downloads $$$ chmod +x install_sw4stm32_linux_64bits-v2.9.run 
steve@McBain:~/Downloads $$$ ./install_sw4stm32_linux_64bits-v2.9.run
```
IMO, change the stupid default installation path from the top of the home directory to
something sane, like `/home/steve/install` or `/opt/software`.  
  
Note the install does add udev rules to `/etc/udev/rules.d`.  
  
Let's get started!  Go to wherever you installed SW4STM32 and run `./eclipse`.  
And... great, I don't see a way to read the flash.  Atmel Studio had this in a separate "Flash Programming" widget.  
I got the hint that SW4STM32 does use OpenOCD so I should be able to find some usable `openocd.cfg`.  
  
Doing a `grep -ir stlink` returns a humourous number of results.  The file we want is not in the STM32Cube-FW_F7... directory.
It is in the SW4STM32 install directory.  And we find it:  
```
steve@McBain:~/installs/sw4stm32 $$$ grep -ir stlink | grep 769
plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board/stm32f769i_disc1.cfg:# This is for using the onboard STLINK/V2-1
plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board/stm32f769i_disc1.cfg:source [find interface/stlink-v2-1.cfg]
plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board/stm32f769i_eval.cfg:# This is for using the onboard STLINK/V2-1
plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board/stm32f769i_eval.cfg:source [find interface/stlink-v2-1.cfg]
plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board/stm32f769i_disco.cfg:# This is for using the onboard STLINK/V2-1
plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board/stm32f769i_disco.cfg:source [find interface/stlink-v2-1.cfg]
```
The files ending with `disc1` and `disco` are identical so I'll pick either one.  
   
Oh no, my `apt install openocd` install did not include config for the STM32F7 : (  Note below there is F4 config but not F7.
```
steve@McBain:~/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts $$$ openocd -f st_board/stm32f769i_disco.cfg
Open On-Chip Debugger 0.9.0 (2018-01-24-01:05)
Licensed under GNU GPL v2
For bug reports, read
  http://openocd.org/doc/doxygen/bugs.html
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
Error: flash driver 'stm32f7x' not found
steve@McBain:~/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts $$$ find /usr/share/openocd/ -name stm32f7*
steve@McBain:~/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts $$$ find /usr/share/openocd/ -name stm32f4*
/usr/share/openocd/scripts/board/stm32f4discovery.cfg
/usr/share/openocd/scripts/board/stm32f429discovery.cfg
/usr/share/openocd/scripts/target/stm32f4x_stlink.cfg
/usr/share/openocd/scripts/target/stm32f4x.cfg
```
  
Ah crap, more bad news from <https://www.carminenoviello.com/2015/07/13/started-stm32f746g-disco/>  :  
```
As I've said at the beginning of this tutorial, the current stable release of OpenOCD (0.9) can't be used to flash STM32F7 processors, since they have a different internal flash from other STM32 MCUs, which requires a different programming procedure. Fortunately, Rémi PRUD'HOMME (I think an ST engineer) has submitted a patch to OpenOCD to enable support STM32F7 family and STM32F746-DISCO board. This patch has been merged in the development OpenOCD repo. So, we can compile a custom version easily. However, I'll not give instructions on how to compile OpenOCD on the Windows platform (read this comment to download a precompiled patched version). Please, check OpenOCD instructions about this.
```
  
Ah yes, good.  The `apt install openocd` version I got was `Open On-Chip Debugger 0.9.0 (2018-01-24-01:05)` , and the one bundled with SW4STM32 was
```
steve@McBain:~/installs/sw4stm32/plugins/fr.ac6.mcu.externaltools.openocd.linux64_1.23.0.201904120827/tools/openocd/bin $$$ ./openocd -v
Open On-Chip Debugger 0.10.0+dev-00021-g524e8c8 (2019-04-12-08:33)
Licensed under GNU GPL v2
For bug reports, read
  http://openocd.org/doc/doxygen/bugs.html
```
  
And now for the annoyingly long command line:  
```
steve@McBain:~/installs/sw4stm32/plugins/fr.ac6.mcu.externaltools.openocd.linux64_1.23.0.201904120827/tools/openocd/bin $$$ sudo ./openocd -f stm32f769i_disco.cfg -s /home/steve/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board -s /home/steve/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/
Open On-Chip Debugger 0.10.0+dev-00021-g524e8c8 (2019-04-12-08:33)
Licensed under GNU GPL v2
For bug reports, read
  http://openocd.org/doc/doxygen/bugs.html
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 2000 kHz
srst_only separate srst_nogate srst_open_drain connect_assert_srst
adapter_nsrst_delay: 100
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 2000 kHz
Info : STLINK v2.1 JTAG v29 API v2 M18 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 3.228947
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : Stlink adapter speed set to 1800 kHz
Info : STM32F769.cpu: hardware has 8 breakpoints, 4 watchpoints
Info : Listening on port 3333 for gdb connections
```
  
And the glorious red/green blinking LED sequence is seen on the discovery kit, and I can issue a reset halt command!  Yay, moving on.  
  
**Reading the flash, writing the flash, verifying the flash, building the Demo source and flashing it.**
  
```
telnet localhost 4444
> flash read_bank 0 /tmp/flashbank0
flash size probed value 2048
wrote 2097152 bytes to file /tmp/flashbank0 from flash bank 0 at offset 0x00000000 in 20.523928s (99.786 KiB/s)
```
  
Indeed, we've read 2M of flash.  Here goes nothing!!!  
Yessss, I did not brick the board.  
  
```
> halt
target halted due to debug-request, current mode: Thread 
xPSR: 0x41000000 pc: 0x0806b398 psp: 0x20013a98
> flash erase_sector 0 0 last
erased sectors 0 through 11 on flash bank 0 in 8.665847s
> reset
``` 
  
First halt the target, then erase all of bank 0, then reset.  The touchscreen becomes unresponsive and the image frozen.  Resetting does not update the image.  Resetting immediately sends the target into a HardFault.  Good. 

```  
> flash write_bank 0 /tmp/flashbank0     
target halted due to breakpoint, current mode: Handler HardFault
xPSR: 0x61000003 pc: 0x20000084 msp: 0xffffffd8
wrote 2097152 bytes from file /tmp/flashbank0 to flash bank 0 at offset 0x00000000 in 22.732296s (90.092 KiB/s)
> 
> reset
```
  
And the board is back online with the same application!
  
Now let's make sure the binary we have from ST matches the one that is loaded on the target.  
We have 2 binaries included here:  
```
steve@McBain:~/prog/embedded/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Binary $$$ l
total 143M
-rw-rw-r-- 1 steve steve  27M Feb  7  2019 STM32CubeDemo_STM32769I-DISCO_V1.4.0.hex
-rw-rw-r-- 1 steve steve 117M Feb  7  2019 STM32769I-DISCO_DEMO_V1.0.0_FULL.hex
```
  
The one ending with `_FULL` is the one preloaded, which has extra 3rd party stuff.  The other one is for the included source code project.  
  
Hmm having some issues with `hex2bin` and `objcopy`, unable to verify the HEX equals the binary.  
The HEX file is 117M big, meaning it must be storing some data like images or video.  
Perhaps that threw off the hex2bin conversion.  
  
Anyway I'm going to move on and now try to build the provided demo project and flash it.  
  
Jeez, the OpenOCD interface is buggy AF!  I have to manually power cycle the board every time.
It takes 50 seconds to flash wow.  Putting the target 1 USB hub deep instead of 2 helps.  
Should not have put it 2 hubs deep.  
  
OK now I'm reasonably in control of the target.  The first experiment I did was to remove some of the modules (ie. "apps"), and see that I can rebuild, reflash and see the difference on the screen.
  
This should be a good starting point.  My plan is to copy one of the modules and use it as
the starting point for my new module, an audio processor module.  It'll use pieces of the
Audio Recorder and Audio Playback modules.  Eventually I'll have to dig deeper in the
example applications to get stuff like USB connectivity, GPIO, etc.
  
Oh goodness I am raging, this tool is a joke.  I'll never let an IDE build my code ever again!!  
Does a clean build almost every time, taking nearly 2 minutes.  
The .project file does all sorts of filesystem crap.  Copying stuff into an "Application" folder?  Wat?  
Thank goodness I use git, otherwise I wouldn't be able to see the crap that's happening behind my back. Oh my god, randomly adding ^M carriage returns.  (this could be an eclipse issue).
  
Literally I can't even make sense of what happens when I right click, "Add New Source Folder" , "Add New Source File", copy paste an existing source file in there.  The include directories are not the same.  I will never depend on a shit IDE (ie. not IntelliJ** for my build configuration, EVER!  
  
  
To free myself from this absolutely egregious vendor lock-in, I will run a clean build and save all of the console output.  This will give me an idea of what compiler flags, include directories, static libraries, etc are used.  Generally I wouldn't need this information but I have negative trust here, in that I trust that there will be tricks and booby traps in this codebase.  

  

# Notes from the depths of the build system
  
**FreeRTOS error: **  :  `error: macro "osThreadDef" passed 5 arguments, but takes just 4` when called like this:  `osThreadDef(osAudio_Thread, Audio_Thread, osPriorityRealtime, 0, 2048);`
  
Knowing it's a macro, I can search for it like this: 
```
grep -ir osthreaddef | grep define
...
Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/cmsis_os.h:#define osThreadDef(name, priority, instances, stacksz) \
Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS/cmsis_os.h:#define osThreadDef(name, thread, priority, instances, stacksz)  \
```
Alright, what is the deal with CMSIS_RTOS_V2?  Clearly we should be using the V2 one, since our code is passing 5 arguments to the macro.  How do we select which one?  Why are both versions included in the codebase?  
  
The problem is clear, I included all directories in the codebase as include directories, which means both CMSIS_RTOS_V2/ and CMSIS_RTOS/ were included, so it's a race to match the file named `cmsis_os.h`.  Thankfully it is clear which is the correct one.  But, the directory structure is not clear.  If I include directories under `Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS` then I might be missing include files under `Middlewares/Third_Party/FreeRTOS/`.  Let's inspect the directory tree.  
Oh gawd, not pretty but it makes sense.  I'll need to manually pick out the headers I'm using, for the CMSIS_RTOS_V2 and also the GCC/ARM7/portmacro.h.  Fine.  

Dang, I missed another definition of that macro.  Here are all the files it's defined in:
```
Drivers/CMSIS/RTOS/Template/cmsis_os.h:#define osThreadDef(name, priority, instances, stacksz)  \
Drivers/CMSIS/RTOS2/Template/cmsis_os.h
Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS/cmsis_os.h
Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/cmsis_os.h
```
And, of course, the ones under Drivers/ are included due to my recursive file globbing.  But which one is correct now?  I want the one with 5 args.  
   
OK I now feel the need to inspect the include directories from the SW4STM32 build.
  
Using this script:
```
import Data.List
import System.Environment

main = do
  args <- getArgs
  if length args /= 1
    then putStrLn "Need 1 filename argument"
    else do

  fileContent <- readFile $ head args
  let 
    tokens = filter (isPrefixOf "-I") (words fileContent)
  
  mapM putStrLn tokens
  return ()
```
We get this:
```
steve@McBain:~/prog/haskell/gcc-commandline-tools $$$ runhaskell extract-include-dirs-from-gcc-commandline.hs /tmp/blah
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Config"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Core/Inc"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Gui/Target"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Gui/STemWin_Addons"
-I"/topdir/Utilities/CPU"
-I"/topdir/Utilities/JPEG"
-I"/topdir/Drivers/CMSIS/Device/ST/STM32F7xx/Include"
-I"/topdir/Drivers/STM32F7xx_HAL_Driver/Inc"
-I"/topdir/Drivers/BSP/STM32F769I-Discovery"
-I"/topdir/Drivers/BSP/Components"
-I"/topdir/Drivers/BSP/Components/Common"
-I"/topdir/Middlewares/ST/STM32_USB_Host_Library/Core/Inc"
-I"/topdir/Middlewares/ST/STM32_USB_Host_Library/Class/MSC/Inc"
-I"/topdir/Middlewares/ST/STM32_USB_Device_Library/Class/MSC/Inc"
-I"/topdir/Middlewares/ST/STemWin/inc"
-I"/topdir/Middlewares/Third_Party/FatFs/src"
-I"/topdir/Middlewares/Third_Party/FatFs/src/drivers"
-I"/topdir/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM7/r0p1"
-I"/topdir/Middlewares/Third_Party/FreeRTOS/Source/include"
-I"/topdir/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS"
-I"/topdir/Middlewares/Third_Party/LwIP/src/include"
-I"/topdir/Middlewares/Third_Party/LwIP/system"
-I"/topdir/Middlewares/Third_Party/LwIP/system/OS"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Modules/Common"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Modules/audio_player"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Modules/audio_recorder"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Modules/videoplayer"
-I"/topdir/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Modules/vnc_server"
-I"/topdir/Drivers/CMSIS/Include"
```
This is less than I thought, so it is managable.  Indeed we see the includes come from under /Middlewares, and there
is nothing from under /Drivers.  Well, since I have this view now, I might as well just "span the space" here,
getting more specific when ambiguities arise.  
  
# The next flaming hurdle
Getting the include directories exactly matching did help.  The next issue is with source files, and it is a rager.  
Remember I was complaining about the `Application/` directory appearing out of the smoke-and-mirrors build system?  Well here's another one:  The `Config/` directory which is in the source, does not appear in the build system!  Moreover, some of the files in `Config/` are built and some arent.  Moreover, the ones that are built are scattered in other places.  How do they get there?  I can't believe someone actually went through the trouble to obfuscate this by hand inside of Eclipse.  
  

Remember how I saved the original build output?  Know how in general the last step of building is the linker phase, where all the object files and static libs are linked together?  Well they did a good job obfuscating it by dumping the list of object files into a file called `objects.list`.  I also saved that file.  So now this is the new source of truth.  But the problem is I can't copy paste the list of object files into my build system like I did with the include directories, because the directory structure is mangled, namely the extra `Application/` and the removed `Config/`.  Moreover the list of object files is kinda long, 163.
  
  
OK let's do some manual labor and instead of file globbing for all sources, let's start from where `main.c` is and see where that takes us.  

# YAFH:  (Yet Another) Flaming Hurdle:  Floating Point ABI's
  
The rage continues, though atleast this time I learned something.  After the above step, 
I had a set of source files all compiling and then I attempted to link, expecting errors.  
But I got something odd:  

`/usr/lib/gcc/arm-none-eabi/4.9.3/../../../arm-none-eabi/bin/ld: error: CMakeFiles/steve-stm32f769-discovery.dir/Core/Src/k_menu.c.o uses VFP register arguments, bin/steve-stm32f769-discovery-0.0.2_copypaste-flags_b9fe442.elf does not`
  
OK.. I could be wrong here but here goes.  The VFP register arguments is what we want, as specified by the compiler option `-mfloat-abi=hard`.  But it says the ELF file doesn't, which baffles me because the ELF file hasn't been linked together yet!  Checking my linker flags 20 times, they have exactly the same flags as the SW4STM32 generated compiler invocations.  
Then I did an experiment.  Instead of using CMake to nicely organize flags, I deleted all that and literally copy pasted the flags from the SW4STM32 compiler and linker invocations.  Sure enough I got the same error.  How???  
  
Then I read a few things on the Internet:  
From <https://github.com/RIOT-OS/RIOT/issues/2660> : 
`Your installed newlib version is built for soft float, the RIOT application is built for hard float`
`This is still an issue in August 2019 with Ubuntu 19.04; is there a reason that the official embedded arm packages don't correctly work with hardfp?`  
  
From <https://stackoverflow.com/questions/9753749/arm-compilation-error-vfp-registered-used-by-executable-not-object-file> :  
`Your target triplet indicates that your compiler is configured for the hard-float ABI. This means that the libgcc library will also be hardfp`  
  
And then it dawned on me... it's my compiler!  
To nail it in, I did this: 
```
steve@McBain:~/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/build $$$ sudo find / -name arm-none-eabi-gcc
...
/home/steve/installs/sw4stm32/plugins/fr.ac6.mcu.externaltools.arm-none.linux64_1.17.0.201812190825/tools/compiler/bin/arm-none-eabi-gcc
/usr/bin/arm-none-eabi-gcc
```
  
I was angry, releived, surprised and not surprised at the same time.  At this point I'm nearly 100% 
sure if I use that other compiler it'll work.  Sure enough, it did.  Yay!  
  

# Linking in the Startup Code
When you get a successful build but the output looks like this:  
```
/home/steve/installs/sw4stm32/plugins/fr.ac6.mcu.externaltools.arm-none.linux64_1.17.0.201812190825/tools/compiler/bin/../lib/gcc/arm-none-eabi/7.3.1/../../../../arm-none-eabi/bin/ld: warning: cannot find entry symbol Reset_Handler; defaulting to 0000000008000000
size /home/steve/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/build/bin/steve-stm32f769-discovery-0.0.2_adding_flags_414d0c6.elf
   text	   data	    bss	    dec	    hex	filename
     88	      8	  16412	  16508	   407c	/home/steve/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/build/bin/steve-stm32f769-discovery-0.0.2_adding_flags_414d0c6.elf
```
The 88 byte text section should be screaming, and of course above it is the hint.  No Reset_Handler means main isn't called, so why bother linking in anything?  
  
Well that was finally a step with no problems.  I just added in the `.s` file.  Didn't specify any flags, hope that doesn't matter.  And yes, the glorious pile of undefined references dumps on the screen.  Ideally, if we resolve them we might be done!  
  
# Listing all the Source Files to Build
  
As I mentioned there are 163 source files to compile.  Thankfully there aren't many directories
that they all live in.  For example there's about 60 sources files in `STM32F7xx_HAL_Drivers/`.  
So I hopefully I can just list a reasonable number of directories and glob everything under them.  
  
There's something very fishy about the directory named `Drivers/STM32F7xx_HAL_Driver`.
The `objects.list` has object files under eg.  `Drivers/STM32F7xx_HAL_Drivers/stm32f7xx_hal.o`.
Notice the trailing `s` as in `Drivers`.  This has to be a booby trap, as doing a grep for
Drivers with an `s` only shows `.project` files.  Not cool!  
  
So adding all of the source files under that `Driver` directory, I get... you guessed it, multiple definitions!  Yet another needle in the haystack to pick out.  Well, after globbing all the files 
in that directory, I don't even know if I can remove single files.  
  
Alright, change of plans then, in the interest of moving this forward.  I will do some VIM-foo to
process `objects.list`, correcting the paths.
  
Wow, there's a spelling error in `objects.list`:  `Application/User/Gui/audio_recoder/audio_recorder_win.o`.  Recoder?  Either it's a French person mispelling it, which does happen, or it's yet another obfuscation.  I'll give them the benefit of the doubt here.  
Actually I take that back, I see some case switching, eg.  `Audio_recorder` to `audio_recorder`.  
`Audio_recorder` doesn't make any sense as a capitalization convention.  
  
Yet more... `Drivers/BSP/Component` to `Drivers/BSP/Components`  
Oh man this one was REALLY hard to find: `Drivers/BSP/STM32769I-Discovery/stm32f769i_discovery.c` from `Drivers/BSP/STM32F769I-Discovery/stm32f769i_discovery.c`.  Can you see the difference?  
It's the `F`.  Why would they remove the `F` besides deliberately obfuscating the build?  Now I'm pretty convinced, I have to say.  
  
This is.... WAAAAAY worse than Atmel ASF, atleast from a structure standpoint.  I haven't looked inside the source code yet.  (Note, ASF source code is crap IMO).
  
Yet more... some of the original codebase has `Src/` , others `src/`.  bah.  
  
# Done specifying source files.  Almost done.
Now that I've compiled all the source files I need, I get to the linker step, which of course
is failing.  I get a bunch of undefined references, and it seems that they are all references
to the static libraries for `STemWin`.  But I don't see why they'd be undefined references because
I did link in the static library.  
Another issue which I will handle first is a multiple definition error:  `CMakeFiles/steve-stm32f769-discovery.dir/startup_stm32f769xx.s.o:(.isr_vector+0x0): multiple definition of `g_pfnVectors'
/home/steve/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Gui/STemWin_Addons/STemWin_Addons_CM7_wc32.a(startup_stm32h743xx.o):(.isr_vector+0x0): first defined here
CMakeFiles/steve-stm32f769-discovery.dir/startup_stm32f769xx.s.o: In function `MDIOS_IRQHandler':
(.text.Default_Handler+0x0): multiple definition of `Default_Handler'
/home/steve/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Gui/STemWin_Addons/STemWin_Addons_CM7_wc32.a(startup_stm32h743xx.o):C:\Users\bennacef\Desktop\SVN\STM32_emWin_Library_SVN\Branches\ST_Addons_binary_Generator\Firmware\Project\ST_Addons_20150406\SW4STM32\ST_Addons\Debug/..\../startup_stm32h743xx.s:127: first defined here`  
  
OK this error message is screaming fishy.  First of all, why does the static library for STemWin, a graphics library, define the vector table and Default_Handler?  
Moreover 100 times, why the hell is the definition inside `startup_stm32h743xx` ?  Why is that startup code inside the library at all?  Why only for 1 variant of microcontroller?!  
`bennacef` , I'm sure you're a fine individual but this work smells like fish.  

But now I'm confused.  I thought if there was a multiple definition in a static library, it would be ignored instead of tripping the multiple def error?
Hmm, maybe if I put the `-lmylib` switches at the end of the linker invocation?  
Ah, that was it.  I put the `-l` switches at the end, and then corrected for yet another naming convention error.  
Static libraries like X.a don't work well with CMake.  CMake wants it like `libX.a`.  
Great!  Now I only have like 10 undefined references. 
  
# LOL, coupling app into kernel.  A joke?
This is hilarious, see this error `CMakeFiles/steve-stm32f769-discovery.dir/Core/Src/k_bsp.c.o: In function `k_TouchUpdate':
/home/steve/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Core/Src/k_bsp.c:155: undefined reference to `SelLayer'`.  
Hmm, what is SelLayer?  
`steve@McBain:~/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0 $$$ grep -r SelLayer  
  
  Projects/STM32F769I-Discovery/Demonstrations/STemWin/Gui/Core/home_alarme/home_alarm_win.c:extern uint8_t   SelLayer;
Projects/STM32F769I-Discovery/Demonstrations/STemWin/Core/Src/k_bsp.c:extern uint8_t SelLayer;
Projects/STM32F769I-Discovery/Demonstrations/STemWin/Core/Src/k_bsp.c:    TS_State.Layer = SelLayer;
Projects/STM32F769I_EVAL/Demonstrations/STemWin/Gui/Core/videoplayer/video_player_win.c:uint8_t                          SelLayer = 0;

  `
So basically if I decide I don't want to build the `video_player` app, the kernel BSP won't compile.  That's just hilarious!  

  
# #include C files for the purposes of obfuscation.  Nice try!
Looking at this error message:  `/home/steve/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0/Projects/STM32F769I-Discovery/Demonstrations/STemWin/Core/Src/k_menu.c:140: undefined reference to `bmF7Logo'`,  
I did a grep for `bmF7Logo`, expecting to find a source file to add to the build.  

`steve@McBain:~/prog/embedded/STM32F769-Discovery/STM32Cube_FW_F7_V1.15.0 $$$ grep -ir bmf7logo`  
`Projects/STM32F769I-Discovery/Demonstrations/STemWin/Gui/Core/settings/settings_res.c:GUI_CONST_STORAGE GUI_BITMAP bmF7Logo = {
Projects/STM32F769I-Discovery/Demonstrations/STemWin/Core/Src/k_menu.c:extern GUI_CONST_STORAGE GUI_BITMAP bmF7Logo;`
  
OK so there's a file called `settings_res.c`.  I could go and add that to the build, but now
that I know how the `.project` level obfuscation works, I'll look inside the `.project` file.  
Searching for `settings` strings, I see this:  
`
<link>
      <name>Application/User/Gui/settings/settings_win.c</name>
      <type>1</type>
      <location>PARENT-2-PROJECT_LOC/Gui/Core/settings/settings_win.c</location>
    </link>
`  
Oh no, I only see `settings_win.c`, where is `settings_res.c` ???  
I was stumped until I looked inside `settings_win.c`.  What do we see at the top ?  

`/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "settings_res.c"
`  
Hah, nice try!
  
# Last little trick
The file named stm32f769i_discovery_audio.c appears in both the top level codebase and also inside
the project directory.  Only one of them is built.  
  
# Yay, I get a build!
  
I'm feeling lucky, let's just flash it.  Let's use GDB to flash the ELF file,
so we don't have to worry about generating .bin binaries.
  
# Closing remarks
?
