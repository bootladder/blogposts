---
layout: post
title: Growing and Maintaining Legacy Code
summary: Don't go bushwhacking the code, do it surgically.
date: 2018-12-29
categories:
    - Embedded
    - C/C++
---
I'm ready to start cleaning up my code and adding new features.
I want to remove all compiler warnings, comments, cruft,
do some obvious refactorings, clean up dependence on common code,
factor out more common code, build all variants under 1 build system
in exactly the same way.  
  
But I don't have unit tests, regression tests, even a manual end-to-end system test is non trivial due to the large, distributed nature of this particular system.  
  
Things that could go wrong:
* Hardware Init fails
* Persistent memory areas corrupted or pointers to it corrupted
* Vector table vectors missing or corrupted
  
Point being, I'm more concerned with system stuff than app stuff.  
Probably the biggest mistake I made was not locking down the system
and the build and never touching it again.  
  
The system is what breaks.  The system is what you don't have.  Hardware.
  
# Solution Component #1: (P)ower (O)n (S)elf (T)est
This is a simple application that exercises the entire system.  
With a test harness this can be automated but manual is a good start.  
Check stuff like:  
* Smoke test
* Clocks
* GPIO 
* Non-volatile memory access, IO bus access
* Peer to Peer comm link
  
The goal is to be confident that if a piece of hardware passes POST,
then app development can **continuously integrate and deliver**.  
  
The combination of POST with app unit tests should be able to catch most bugs immediately after being introduced.  

  
Though an automated test harness requires extra hardware to probe and read IO on the DUT, we can get 1 step closer with a automatic **continuous deployment** of the latest build, so you can `git push` your code, it gets built and deployed, then you go and manually verify.  
  
# Problem:  How do you read the results of the POST?  Need some comm link
Serial or wireless , or perhaps write the results in Flash.  
Well crap, this particular variant doesn't have a UART driver although I could easily drop one in.  
I don't have a sniffer setup, although I could set one up.  
But really a UART is mandatory.  How else are you going to unit test on the target?

