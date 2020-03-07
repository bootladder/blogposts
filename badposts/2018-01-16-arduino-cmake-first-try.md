---
layout: post
title: arduino-cmake first try
---
I cloned the repo.
```
cd example
mkdir build
cmake ..
CMake Error at CMakeLists.txt:11 (GENERATE_ARDUINO_LIBRARY_EXAMPLE):
  Unknown CMake command "GENERATE_ARDUINO_LIBRARY_EXAMPLE".
```
OK `https://github.com/queezythegreat/arduino-cmake/issues/38` solves it.  
mkdir build;cmake .. must happen in the top level CMakeLists.txt directory.  
```
steve@steve-ThinkPad-T420:~/prog/scratch/arduino-cmake$ mkdir build
steve@steve-ThinkPad-T420:~/prog/scratch/arduino-cmake$ cd build
steve@steve-ThinkPad-T420:~/prog/scratch/arduino-cmake/build$ cmake ..
CMake Error at cmake/ArduinoToolchain.cmake:93 (message):
  Could not find Arduino SDK (set ARDUINO_SDK_PATH)!
Call Stack (most recent call first):
  /usr/share/cmake-3.5/Modules/CMakeDetermineSystem.cmake:98 (include)
  CMakeLists.txt:15 (project)
```
OK, it can't find the toolchain.  It's not installed.  What happened ...  
Line 15 of the top level CMakeLists.txt, was a project() call.  
```
project(ArduinoExample C CXX ASM)
```
And then project() calls CMake core... line 98
```
     include("${CMAKE_TOOLCHAIN_FILE}" OPTIONAL RESULT_VARIABLE _INCLUDED_TOOLCHAIN_FILE)
```
`CMAKE_TOOLCHAIN_FILE` is set in the CMakeLists.txt: 
```
#=============================================================================#
# Author: QueezyTheGreat                                                      #
# Date:   26.04.2011                                                          #
#                                                                             #
# Description: Arduino CMake example                                          #
#                                                                             #
#=============================================================================#
set(CMAKE_TOOLCHAIN_FILE cmake/ArduinoToolchain.cmake) # Arduino Toolchain


cmake_minimum_required(VERSION 2.8)
#====================================================================#
#  Setup Project                                                     #
#====================================================================#
project(ArduinoExample C CXX ASM)

print_board_list()
print_programmer_list()

add_subdirectory(example)   #add the example directory into build
~                                                                     
```
So we are reading the ArduinoToolchain.cmake. 
Some interesting things inside here, some I don't understand.  
```
set(CMAKE_SYSTEM_NAME Arduino)

set(CMAKE_C_COMPILER avr-gcc)
set(CMAKE_ASM_COMPILER avr-gcc)
set(CMAKE_CXX_COMPILER avr-g++)
``` 
Here we are identifying avr-gcc, that's cool.  
```
#=============================================================================#
#                         System Paths                                        #
#=============================================================================#
if (UNIX)
    include(Platform/UnixPaths)
```
include() is like #include in C, so let's see it.  Huh.. it's not there.  Let's see later then...  
  
Then there's a part later to detect the `ARDUINO_SDK_PATH` using hints, which are directories likely to have the install.  
Let's just install the SDK now.  To do that just download it and run `sudo install.sh`  

Now let's run `cmake ..` again.  Still the same error, let's check the path hints.  
Hmm for some reason the install.sh  doesn't actually install anything in /usr or /bin or whatever.
Let's move it manually then into one of the `Hints` , /usr/local/share/ .  
**Awesome!**
```
steve@steve-ThinkPad-T420:~/prog/scratch/arduino-cmake/build$ cmake ..-- The C compiler identification is GNU 4.9.2
-- The CXX compiler identification is GNU 4.9.2
-- The ASM compiler identification is GNU
-- Found assembler: /usr/local/share/arduino-1.8.5/hardware/tools/avr/bin/avr-gcc
-- Arduino SDK version 1.8.5-2017.01.09: /usr/local/share/arduino-1.8.5
-- Check for working C compiler: /usr/local/share/arduino-1.8.5/hardware/tools/avr/bin/avr-gcc
-- Check for working C compiler: /usr/local/share/arduino-1.8.5/hardware/tools/avr/bin/avr-gcc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/local/share/arduino-1.8.5/hardware/tools/avr/bin/avr-g++
-- Check for working CXX compiler: /usr/local/share/arduino-1.8.5/hardware/tools/avr/bin/avr-g++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- ARDUINO Boards:
--                   menu: 
--                    yun: Arduino Y
--                    uno: Arduino/Genuino Uno
--              diecimila: Arduino Duemilanove or Diecimila
--                   nano: Arduino Nano
--                   mega: Arduino/Genuino Mega or Mega 2560
--                megaADK: Arduino Mega ADK
--               leonardo: Arduino Leonardo
--            leonardoeth: Arduino Leonardo ETH
--                  micro: Arduino/Genuino Micro
--                esplora: Arduino Esplora
--                   mini: Arduino Mini
--               ethernet: Arduino Ethernet
--                    fio: Arduino Fio
--                     bt: Arduino BT
--             LilyPadUSB: LilyPad Arduino USB
--                lilypad: LilyPad Arduino
--                    pro: Arduino Pro or Pro Mini
--               atmegang: Arduino NG or older
--           robotControl: Arduino Robot Control
--             robotMotor: Arduino Robot Motor
--                  gemma: Arduino Gemma
--     circuitplay32u4cat: Adafruit Circuit Playground
--                yunmini: Arduino Y
--                chiwawa: Arduino Industrial 101
--                    one: Linino One
--                unowifi: Arduino Uno WiFi
-- 
-- ARDUINO Programmers:
--            avrisp: AVR ISP
--        avrispmkii: AVRISP mkII
--        usbtinyisp: USBtinyISP
--        arduinoisp: ArduinoISP
--     arduinoisporg: ArduinoISP.org
--            usbasp: USBasp
--          parallel: Parallel Programmer
--      arduinoasisp: Arduino as ISP
--          usbGemma: Arduino Gemma
--         buspirate: BusPirate as ISP
--            stk500: Atmel STK500 development board
--          jtag3isp: Atmel JTAGICE3 (ISP mode)
--             jtag3: Atmel JTAGICE3 (JTAG mode)
--         atmel_ice: Atmel-ICE (AVR)
-- 
-- Generating wire_example
-- avr library found: Wire
-- Generating blink_example
-- Generating blink_original
-- Generating blink_file_original
-- Generating blink_bundled
-- Generating blink_lib
-- Generating blink
-- Generating serial_lib
-- avr library found: SoftwareSerial
-- Configuring done
-- Generating done
-- Build files have been written to: /home/steve/prog/scratch/arduino-cmake/build
steve@steve-ThinkPad-T420:~/prog/scratch/arduino-cmake/build$ 

```

What's that stuff about boards and programmers?
How does that last part work, `Generating blink_example` ?  
Somehow, CMakeLists.txt at the top level was able to find all of those things.  

**Easy** , that's in the top CMakeLists.txt:  
```
#====================================================================#
#  Setup Project                                                     #
#====================================================================#
project(ArduinoExample C CXX ASM)

print_board_list()
print_programmer_list()

add_subdirectory(example)   #add the example directory into build
```
So literally it prints the board lists and programmer list, and recurses into the example directory.  
  
So, the examples/CMakeLists.txt.  
All that happens is this:
```
set(ARDUINO_DEFAULT_BOARD uno) # Default Board ID, when not specified
set(ARDUINO_DEFAULT_PORT /dev/ttyUSB0) # Default Port, when not specified
```
```
#====================================================================#
# Advanced firwmare example
#====================================================================#
generate_arduino_firmware(blink
        SRCS blink.cpp
        LIBS blink_lib
        BOARD nano
        BOARD_CPU atmega328
        PORT com4
        )
```
Wow that's not too bad , I'd like to give it a try!  
  
It looks like it should be easy to support the ARM chips, and use this to create new Arduino Libraries. 

