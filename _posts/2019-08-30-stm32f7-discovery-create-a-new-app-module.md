---
layout: post
title: 'STM32F7 Discovery:  Create a new app module'
---
Now that I got the build working under CMake, the next thing to do is create my own app module.  
My plan is to combine pieces from the other modules to create a new module,
then start stripping everything else out of the build so it builds faster, is easier to modify,
runs faster, etc.
  
The module I'm starting with is `audio_recorder_win.c` because it's the smallest
module that includes some functionality that I want.
  
At the top level I do like how it looks:  
```
in main:
k_ModuleAdd(&steve_board);

in steve_win.c:
K_ModuleItem_Typedef  steve_board =
{
  3,
  "Audio Recorder   ",
  open_recorder,
  0,
  Startup,
  NULL,
}
;
```
  
I started by copy pasting files and changing the name.  I get about 20 multiple definitions
as expected.  Hopefully can start learning something about how the GUI framework works.  
  
# 1.  _cbNotifyStateChange() .  The only non-static _cb function.  Why?  

`steve_win.c` has this callback, and `audio_recorder_app.c` calls it in 2 places,
```
AUDIO_RECORDER_StopRec(void)
AUDIO_RECORDER_StopPlayer(void)
```
Haha, what is inside the definition of _cbNotifyStateChange() ?  You guessed it,
a reference to `audio_recorder_app.c`.  2 .c files including each others headers... smelly.  
  
Not too bad.  All the other references inside that callback are to the GUI framework.
But it is a namespace collision waiting to happen.
  
I mean really, what's the point of having a kernel at all when you do this stuff?  
  
# Common/audio_if.h  super bad namespace collision
I understand why they factored out audio_if.  It's because the video player and audio player have the same control interface.  
But the symbols defined under the common audio_if have the same prefix as the actual module, AUDIO_RECORDER.  
Because of that, you can't do a blind search/replace of the AUDIO_RECORDER prefix.  
  
# In general, I'd prefix all non-static symbols with the module name
```
void BSP_AUDIO_IN_TransferComplete_CallBack(void)
{
  osMessagePut ( AudioEvent, REC_BUFFER_OFFSET_FULL, 0);  
}

```
I may be judging too early here since this is a callback.  But, the file is 700 lines long.
It's not intiuitive that callbacks would be here with a global namespace.  
My intuition would be to have a layer of indirection to put the callbacks, where it is clear
where the callbacks are called from and where they are handled.  At the expense of possible overhead.  

# Interesting Header File Mistakes
Looking at `k_menu.c` , its only include is `main.h`.  My goal here is to replace `main.h`
with only the headers that are actually needed.  So of course the idea is to delete
`#include "main.h"` and then add in #include's to resolve the symbols.  
I was almost at the end, and then I got a huge pile of errors missing the same definition:  
`/home/steve/prog/embedded/STM32F769-Discovery/stm32f769i-discovery-demo-stemwin/Drivers/HAL/stm32f7xx_hal_rcc_ex.h:3208:1: error: unknown type name 'HAL_StatusTypeDef'; did you mean 'SAI_Block_TypeDef'?
 HAL_StatusTypeDef HAL_RCCEx_PeriphCLKConfig(RCC_PeriphCLKInitTypeDef  *PeriphClkInit);`
   
Wait, `HAL_StatusTypeDef` is defined in `stm32f7xx_hal_def.h`, which I put as the very first
#include in `k_menu.c` !  It almost seems like the symbol was undefined somehow.  But I'm pretty sure
that is not possible.  You can #undef a #define but not a `typedef`.  
  
I sat back and thought, and then I realized it had to be due to the order of includes within the header files.  I looked inside `stm32f7xx_hal_def.h`.  Here is the very top of it:  
```
/* Includes ------------------------------------------------------------------*/
#include "stm32f7xx.h"
   //#include "Legacy/stm32_hal_legacy.h"
#include "stm32_hal_legacy.h"
#include <stddef.h>

/* Exported types ------------------------------------------------------------*/

/** 
  * @brief  HAL Status structures definition  
  */  
typedef enum 
{
  HAL_OK       = 0x00U,
  HAL_ERROR    = 0x01U,
  HAL_BUSY     = 0x02U,
  HAL_TIMEOUT  = 0x03U
} HAL_StatusTypeDef;
```
OK, `HAL_StatusTypeDef` is literally the first thing defined, and it cannot be undefined,
so how could there be `unknown type name` ??  
Well, using logic, it must be that the errors happen inside the #includes of this file.  
ie.  one of the other #included header files is using the symbol `HAL_StatusTypeDef`.  

Looking at `stm32_hal_legacy.h`, it doesn't #include anything so the problem is in `stm32f7xx.h`.  
And that's another huge catch-all header file.  Here's the fishy part:  if you follow
the include chain from there, it goes back to `stm32f7xx_hal_def.h`!  Circular dependency!  
Of course this is why we have #include guards here.  
And that is the clue.  The #include guard was #defined, and the #include of other header files appear before the symbol `HAL_StatusTypeDef` was defined.  So, deep inside the include chain here will
be unknown symbols even though the symbol is defined at the beginning of the include chain!  Wow!  
  
As a sanity check, let's look at the include chain from `main.h`.  
Nice, I am not insane after all!  I mean, the circular include is 7 includes deep, give me a break haha.  
To illustrate, I'll show the include chain from `main.h`.  
``` 
main.h
  stm32f7xx_hal.h
    stm32f7xx_hal_conf.h
      stm32f7xx_hal_rcc.h
        stm32f7xx_hal_def.h
         stm32f7xx.h
           stm32f7xx_hal.h  --this one is guarded
```
Recall the symbol in question (HAL_StatusTypeDef) is defined in `stm32f7xx_hal_def.h`,
and the first error is encountered in `stm32f7xx_hal_rcc.h`
But by the time we get there, there are already include guards set, meaning that
the entirety of stm32f7xx_hal_def.h will be included before any of stm32f7xx_hal_rcc.h.
  
Nice!  That resulted in a good tip:  **Include stm32f7xx_hal.h** to get the whole HAL.
Don't include individual HAL files.  We'll fix that issue later.  One level at a time haha!
 
 
# Isolating the Pieces of a Module: Window Manager, RTOS, HAL/BSP
In trying to figure out how the architecture works, I'll split out a new C file from `steve_win.c`,
which will have only Window Manager stuff.  Hopefully the #include list will be small and the external
references will increase understanding.
