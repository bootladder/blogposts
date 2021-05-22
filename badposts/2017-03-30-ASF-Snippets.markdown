---
layout: post
title:  "ASF Snippets that only depend on CMSIS"
description: "IS this the description?"
summary: "is summary??"
categories:
  - "C/C++"
  - "Embedded"
---

I've always had problems getting ASF dependent crap to compile.  But the code works so I have to go to it
either for use or reference.  I just discovered, with slight editing, a lot of the lowest level ASF functions
can be snipped out and compiled using only the CMSIS headers.
  
The motivation for this was: how to read and write to the GCLK registers.  They require writes before reads and writes before writes.
Datasheet doesn't explictly state that but the ASF code does.  I discovered that the special C code to do this register access is not
dependent on any ASF headers!

{% highlight c%}
static bool is_gclk_adc_enabled(void)
{
  bool enabled;

  /* Select the requested generic clock channel */
  *((uint8_t*)&GCLK->CLKCTRL.reg) = 0x13;
  enabled = GCLK->CLKCTRL.bit.CLKEN;

  return enabled;
}
static void set_gclk_gen_5_osc8m(void)
{
  while (system_gclk_is_syncing()) {
    /* Wait for synchronization */
  };

  /* Select the correct generator */
  *((uint8_t*)&GCLK->GENDIV.reg) = 0x05;

  /* Write the new generator configuration */
  while (system_gclk_is_syncing()) {
    /* Wait for synchronization */
  };
  GCLK->GENDIV.reg  = 0x00000205;

  while (system_gclk_is_syncing()) {
    /* Wait for synchronization */
  };
  GCLK->GENCTRL.reg = 0x00010605;

}
static inline bool system_gclk_is_syncing(void)
{
  if (GCLK->STATUS.reg & GCLK_STATUS_SYNCBUSY){
    return true;
  }

  return false;
}

{% endhighlight %}


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[wiki-apage]:  /wiki/todo
