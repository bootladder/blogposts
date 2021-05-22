---
layout: post
title:  "SAMD10 ADC Init, via direct register access"
categories:
  - "C/C++"
  - "Embedded"
---

First: how to brick your device.  
If you write to a write-synchronized register twice, before the 2 clock domains sync,
the bus will stall until the clocks sync.
However, if you haven't initialized the GCLK for that peripheral,
the clocks won't sync and the bus stalls forever.
If you had put the code in shortly after the reset vector is called,
the bus will stall and you can't regain control using the debugger.
  
Note:  there is an unbricking antidote that Atmel supplies.
  
  
Second:  a lesson on 32-bit architecture  
I'm not sure why, but before now I assumed that 32-bit architecture meant that
all reads and writes are 32-bits.  That implied that any 8-bit write involved 
doing a 32-bit read, doing the logic and shifting, then doing a 32-bit write.
  
Well atleast in Cortex-M0+ that is totally false.
The instruction set has loads and stores for byte, halfword, word.
eg ldrb, ldrh, ldr.
  
If you look at the .lss output from arm-none-eabi-gcc, you'll see all the various instructions,
ldrb, ldrh, ldr.  How does GCC know which instruction to use?  Is it entirely based on the type?
  
In other words, does a uint8_t 100% absolutely lead to a ldrb/strb instruction ?
I think if there were a one-to-one mapping from primitive type to the load/store instruction,
that would be good to know!  Who is responsible for this?  C standard?  GCC?  ARM?
  
After some research it is true enough for my purposes.  See this example.  
The C code simply writes the argument to a location that is stored in RAM at a fixed address.  
You can see the strb instruction, ie. the 8-bit write.

{% highlight c%}
uint32_t main(uint32_t arg)
{
  //Get the address to write to, from the global RAM
  uint32_t * ramptr = (uint32_t *) 0x20000100;
  volatile uint32_t writeAddr = *ramptr;

  //Now we have the address in a uint32_t
  //Point to that address and write to it
  volatile uint8_t * writePtr = (volatile uint8_t *) writeAddr;
  *writePtr = (uint8_t)arg;
  return 0xAA;
}
{% endhighlight %}

{% highlight asm%}
00000010 <main>:
  10: 4b04        ldr r3, [pc, #16] ; (24 <main+0x14>)
  12: b082        sub sp, #8
  14: 681b        ldr r3, [r3, #0]
  16: b2c0        uxtb  r0, r0
  18: 9301        str r3, [sp, #4]
  1a: 9b01        ldr r3, [sp, #4]
  1c: 7018        strb  r0, [r3, #0]
  1e: 20aa        movs  r0, #170  ; 0xaa
  20: b002        add sp, #8
  22: 4770        bx  lr
  24: 20000100  .word 0x20000100
{% endhighlight %}
  
  
Now we have the 32-bit write.  Notice the str r0 instruction instead of the strb.  
   
{% highlight c%}
uint32_t main(uint32_t arg)
{
  //Get the address to write to, from the global RAM
  uint32_t * ramptr = (uint32_t *) 0x20000100;
  volatile uint32_t writeAddr = *ramptr;

  //Now we have the address in a uint32_t
  //Point to that address and write to it
  volatile uint32_t * writePtr = (volatile uint32_t *) writeAddr;
  *writePtr = arg;
  return 0xAA;
}
{% endhighlight %}
{% highlight asm%}
00000010 <main>:
  10: 4b04        ldr r3, [pc, #16] ; (24 <main+0x14>)
  12: b082        sub sp, #8
  14: 681b        ldr r3, [r3, #0]
  16: 9301        str r3, [sp, #4]
  18: 9b01        ldr r3, [sp, #4]
  1a: 6018        str r0, [r3, #0]
  1c: 20aa        movs  r0, #170  ; 0xaa
  1e: b002        add sp, #8
  20: 4770        bx  lr
  22: 46c0        nop     ; (mov r8, r8)
  24: 20000100  .word 0x20000100
{% endhighlight %}

  
This is the API I use to write a register.  
You have to explicitly read the register first, then write it.
  
{% highlight rust %}
fn write_register(data: [u8;4])  ->  Vec<u8> {
fn read_register( baseaddr: u32, offset:u8 ) -> Vec<u8>  {
{% endhighlight %}

Here is a link [Jekyll docs][jekyll-docs] to somewhere.  
   
Here is a link to the wiki [a wiki page][wiki-apage] blah blah  

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[wiki-apage]:  /wiki/todo
