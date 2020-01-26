---
layout: post
title: 'STM32F769I-Discovery: Isolating and Porting the Demo'
---
This is the second time porting the STemWin demo from the `STM32Cube_FW_F7_V1.15.0` project.  
I realized that there was no need to keep all the extra files in my codebase.  
Instead, I can just cherry pick out only the files that go into the build.
This way it is much easier to navigate the code when attempting to understand and modify it.
Interestingly this is how the SW4STM32 projects seem to be structured.  
But instead of actually making sense and making copies or symlinks from the original-codebase-for-all-targets to the single project that's being built, they made Eclipse level links, which means I can't just copy paste the filesystem structure somewhere else and re-use the Eclipse-generated makefiles.  That's a mouthful!  
  
Anyway I have a choice now in how to structure the files in this new project.  I could mimic the
structure from the SW4STM32 Eclipse project, but given it smells bad I will stay away.  
Particularly the spelling errors eg. `audio_recoder/audio_recorder_win.o` , and the
terrible lack of naming convention eg. `Audio_recorder/audio_recorder_app.o` and `Video_Player/GUI_AVI.o`.  
But I do like the top level, ie. breaking it out into `Application`, `Drivers`, `Middlewares`.  
  
I'm going to just get it right first and then change structure later.
  
  
Skipping ahead, I've got a working build now.  
Recap, here is how I run OpenOCD on the target:  
```
steve@McBain:~/installs/sw4stm32/plugins/fr.ac6.mcu.externaltools.openocd.linux64_1.23.0.201904120827/tools/openocd/bin $$$ sudo ./openocd -f stm32f769i_disco.cfg -s /home/steve/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/st_board -s /home/steve/installs/sw4stm32/plugins/fr.ac6.mcu.debug_2.5.0.201904120827/resources/openocd/scripts/
```
  
And here is the one liner for flashing the target with GDB
```
steve@McBain:~/prog/embedded/STM32F769-Discovery/stm32f769i-discovery-demo-stemwin/build/bin $$$ arm-none-eabi-gdb steve-stm32f769-discovery-0.0.2_master_76a2068.elf -ex 
"target remote localhost:3333" -ex "load" -ex "monitor reset" -ex "quit"
```
  
Again, the goal this time is to have a cleaner codebase so it'll be easier to refactor and make changes.
  
# Let's make a change!
I want to change the name of the new app, so it shows up on the menu after the system boots up.  
```
... inside steve_win.c
K_ModuleItem_Typedef  steve_board =
{
  4,
  "Steve App",
  Steve_open_recorder,
  0,
  Startup,
  NULL,
}
;
```
Cool, that was easy.  
  
### Now I'm looking at the #include "main.h" in steve_win.c
What is being used from main.h?  I don't like catch-all header files like that one.  
It's easier to use at first when you just want to include everything and not care,
but later it will bite you!  
So let's comment out `#include "main.h"` and see what happens.  
Wow!  The only errors are for colors.  
Nice!  I can just replace `#include "main.h"` with `#include "ST_GUI_Addons.h"`.  
  
Also note `main.h` was included in `steve_win.h`.  Generally a bad smell.  Fortunately
the build still worked after removing it.

### Now I just don't like `main.h` at all.  It reminds me of ASF (atmel).
ASF.h gave me lots of problems, particularly for writing unit tests.  
In my opinion, header files are for exposing public interfaces.  Here, `main.h` is just
a garbage dump for including other header files.
  
But to really get rid of #include'ing `main.h`, you have to find all occurances of
`#include "main.h"`, becuase any header file that has that will bring in all of main.h
```
steve@McBain:~/prog/embedded/STM32F769-Discovery/stm32f769i-discovery-demo-stemwin $$$ grep -r 'include "main.h"' --include=*.h
Modules/steve_app.h://#include "main.h"
Modules/audio_player_app.h:#include "main.h"
Modules/audio_recorder_app.h:#include "main.h"
Modules/audio_if.h:#include "main.h"
Application/k_module.h:#include "main.h"
Application/k_rtc.h:#include "main.h"
Application/k_bsp.h:#include "main.h"
Application/k_storage.h:#include "main.h"   
Utilities/cpu_utils.h:#include "main.h"
Gui/Target/LCDConf.h:#include "main.h"
```
Here, `cpu_utils.h` and the `k_` ones_stand out, as it is being included inside `main.h`.
That's another smell, main.h including a file that includes main.h
