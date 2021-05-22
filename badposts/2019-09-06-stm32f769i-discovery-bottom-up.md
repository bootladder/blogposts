---
layout: post
title: "STM32F769 Discovery: Trying it Bottom Up"
---
After discovering 1. There's a whole set of Application/ and Example/ projects for the
discovery kit, and 2. The Demonstration/ application does too many things so it's confusing...  
I will now try to cross examine all the code I can, and then try to build up a simple app
with building blocks.  

  
Basically I want audio loopback, UART transmit, LCD drawing.
  
It seems that interrupts are problematic.  With multiple levels of ISRs and callbacks,
it's confusing.  A good debugging trick is to put a infinite printing loop at all of the ISR vectors.  
  
It's really annoying how there aren't compiler errors for undefined interrupts.  
I believe this is a mal-use of the `weak` pragma.



