---
layout: post
title: 'CMake: Building an App for Multiple Targets'
---
I have an application, it's a sniffer.  
I want it to run on bare metal, on an OS, and on multiple CPU targets.  
Basically anything with a SPI port and the radio should be able to run this.  
OK, looks like it worked.  Here's what I learned:  
  
```
steve@steve-ThinkPad-T420:~/prog/debos/debos_sniffer$ tree
.
├── build.sh
├── CMakeLists.txt
├── Makefile
├── platform
│   ├── CMakeLists.txt
│   ├── samd20_baremetal
│   └── samd20_debos
│       ├── atsamd20j18.ld
│       ├── CMakeLists.txt
│       └── halStartup.c
├── README.md
├── src
│   ├── app.c
│   ├── my_memcpy.c
│   ├── my_memcpy.h
│   └── System.h
└── tool
    ├── sniffer_parser_lwmesh.py
    └── sniffer_reader.py
```
```
#### The Top Level CMakeLists.txt
cmake_minimum_required(VERSION 3.5)

add_subdirectory(platform)
```
```
#### The platform/ CMakeLists.txt
add_subdirectory(samd20_debos)
```
  
Now, here's where all the original stuff went.  platform/samd20_debos CMakeLists.txt.  I made a few modifications to it.  
  
First notice in the tree, that I have halStartup.c inside of platform/, not src/.  This is because halStartup.c is platform specific; it defines the vector table, startup code, etc.  In the case of the bootloaded (debos) version, the startup code is actually dummy, and it is just there to help the linker create the binary image correctly.  But for the bare metal version, the startup code needs to be correct and actually call the main()  while(1)
  
src/ only has app.c , which currently does depend on a SAMD20 specific library.  We'll take care of that later.

```
#set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/bin)
#set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
#set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})
```
I commented those out, I need to figure out whether I'm supposed to set them myself, or use the preset values.  I did notice that setting CMAKE_BINARY_DIR causes the built binaries to show up there, which is what I want.  

```
 add_custom_command(TARGET debos_sniffer POST_BUILD
-    COMMAND size bin/debos_sniffer
+    COMMAND size debos_sniffer
 )
```
Here's another interesting one.  Before when CMAKE_BINARY_DIR was set to ${CMAKE_SOURCE_DIR}/bin, the custom command worked by specifying bin/debos_sniffer.  This is bin/ at the top level, where .git/ is.  But now, the compiled binary shows up in build/platform/samd20_debos/ , and somehow it is found by just specifying debos_sniffer.

``` 
+#file(GLOB SOURCES
+#    "*.h"
+#    "*.c"
+#    "hal/*.c"
+#    "hal/*.h"
+#)
+
+set(SOURCES   ${CMAKE_SOURCE_DIR}/src/app.c  
+							${CMAKE_SOURCE_DIR}/src/my_memcpy.c 
+	 ) 
+
```
Here you see I replaced file(GLOB SOURCES ...) with set(SOURCES ...),
as recommended by people.  Notice I'm specifying more exactly where the source is; the previous file(GLOB SOURCES ...) worked when CMakeLists.txt was inside the directory with the source.
  
# OK now that I have the original target built and working, I can add another target.  
This is the bare metal target.  It's basically the same thing except:  
App vector table has to be correct; these are Reset Vector and ISRs.  This app doesn't use any ISRs so it can't go too horribly wrong.  
  
To check that the bare metal app is built correct, I will do the following:  
- Check that Reset Vector points to startup code.  
- Check that startup code points to and calls main().  
- Check that main() points to and calls app().  
- Check that the calls to Checkpoint() are stubbed out.
  
# 1. add_subdirectory() in CMakeLists.txt inside platform/ 
# 2. copy halStartup.c, linkerscript.ld, CMakeLists.txt from the working target   
Now let's see what the build says.  Dang it's a tricky one!  
```
CMake Error at /usr/share/cmake-3.5/Modules/ExternalProject.cmake:2405 (add_custom_target):
  add_custom_target cannot create target "samd20_headers" because another
  target with the same name already exists.  The existing target is a custom
  target created in source directory
  "/home/steve/prog/debos/debos_sniffer/platform/samd20_debos".  See
  documentation for policy CMP0002 for more details.
Call Stack (most recent call first):
  platform/samd20_baremetal/CMakeLists.txt:69 (ExternalProject_Add)
```
So, both CMakeLists.txt are identical, so I didn't expect it to work anyway.  We see here it was because of ExternalProject_Add.  
The target "samd20_headers" already exists.  
Well, since the 2 projects use exactly the same library, let's try to move the ExternalProject_Add up a level.  So remove the duplicated calls to ExternalProject_Add and stick it in the CMakeLists.txt 1 level up.
  
Cool, I did that and the I got an error that I was actually expecting.
```
CMake Error at platform/samd20_baremetal/CMakeLists.txt:117 (add_executable):
  add_executable cannot create target "debos_sniffer" because another target
  with the same name already exists.  The existing target is an executable
  created in source directory
  "/home/steve/prog/debos/debos_sniffer/platform/samd20_debos".  See
  documentation for policy CMP0002 for more details.
```
This is obviously the re-use of the executable's name, debos_sniffer.  That has to change.  
Woah cool looks like they both built!  
  
# Let's open up Hopper and see what it looks like
  
For an ARM Cortex M0+ the Reset Vector is located at address 0x4.  
I got a 0xd9220000, and changing from bad-endian to good-endian, I get the address 0x22d9.  What is that...? `HAL_IrqHandlerReset` Yes!  That's my startup code.  Now, does it call main?  Hopper says it does.  Now, does main call app() in an infinite loop?  Yes! 
  
OK now for the tricky part.  The Bootloaded version used a function called Checkpoint() which would take a string and some flags, and use the Bootloader's UART interface to output the bytes, turn on LED, halt the CPU or whatever.  
The app itself doesn't configure any serial out.  Also, the Checkpoint() function doesn't exist so calling it would be bad.  
So, this is not going to be very eventful, since a sniffer literally just outputs serial and nothing else.  
  
# Well just for fun, let's add a LED toggle at the App level.  
To do this, I will bring in some app code that turns on a LED of a different color.  There are 3 LEDs with different colors on the board that I have.  The Bootloader has control of 1 of them.  So let's have the app turn on the other one, that tells us the app is in control.
  
OK that compiled, but now I'm back to my Jenkins issue.  When I push the repo, Jenkins will build it, which will create 2 sets of compiler outputs, one for bootloaded one for bare metal.
  
I want to be able to grab either binary without typing out a long ass pathname.
