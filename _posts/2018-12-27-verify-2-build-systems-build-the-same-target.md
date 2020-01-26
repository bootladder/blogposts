---
layout: post
title: Verify 2 Build Systems Build the Same Target
---
I had a verbose explicit Makefile and ported it to a CMake build.  
After getting past the compile and link errors I saw the output
executable was much smaller than expected.  
How do I know what's missing?  
  
1st:  If the size is significantly different, that's good.  You definitely missed something.  If the size was almost the same it's a more subtle reason.  
  
Now let's go over some technique.  
  
# Scan lists of objects with `nm`
```
nm mything.elf | cut -d' ' -f3 > /tmp/thing1
nm mythig2.elf | cut -d' ' -f3 > /tmp/thing2
```
Now compare them side by side.  The symbols are sorted alphabetically.
In my case I immediately notice lots of missing HAL symbols.  
Either the object files don't exist or they didn't get linked in.  
Pick a missing symbol, figure out which source file defines that symbol, then go looking for that source file's object flie.  
  
Turns out a set of compiler flags were not there, which were all duplicates/overrides, one of which was the optimization setting -O0.  The new, smaller sized build had -Os.  After changing back to -O0 the size was 71232, but I expected 76036.  Dang, where is that missing 5k?  
  
Next clue:  new build gets a lot of compiler warnings.  
  
And turns out, even more compiler flags were not brought over.  
Well unfortunately my diff skills are being challenged.  The gcc
invocations are starting to look like binary files.  
I need a system to analyze gcc flags in command lines,
when the flags can be out of order.  
  
Here's a bit of haskell to list out different compiler flags
```
import Data.List
main = do
  file1 <- readFile "input1.txt"
  file2 <- readFile "input2.txt"
  let
      filterTokens = filter (not . isPrefixOf "-I")
      diffTokens l1 l2 = filter (\x -> not $ elem x l2) l1

      filteredTokens1 = filterTokens $ words file1
      filteredTokens2 = filterTokens $ words file2
      t1 = diffTokens filteredTokens1 filteredTokens2
      t2 = diffTokens filteredTokens2 filteredTokens1

  putStrLn "Tokens in 1, not in 2"
  mapM putStrLn t1
  putStrLn "\n\nTokens in 2, not in 1"
  mapM putStrLn t2
  return ()
```
  
Well, after some more comparing and matching, I did get the expected output, exactly the same size.  
Didn't get any more insights out of it though :(  
And for some reason I'm getting warnings on the new build that I don't get in the old build, though the warning flags seem to match now.

  
# Round 2, build a different target
I repeated the steps for a different build but the old and new build have different size.  
I thought the reason would be due to removing unused code, but actually the new build
is larger than the old by 300 bytes in .text at ~80k total.  Interestingly the
.data section is about 1k bytes larger.  
  
Here's an idea, use the diffing code above to diff the outputs of the `nm` snippets at top.  
Difference being that the `nm` snippets are separated by lines not spaces.
  
Oh wow, this is pretty cryptic
```
steve@steve-T420-ZoZ:~/prog/haskell/gcc-commandline-differ$ runhaskell diffbylines.hs /tmp/nm_valve_orig /tmp/nm_valve_combined 
Tokens in 1, not in 2
AlarmMsg
errno
heap.5211
malloc
__malloc_free_list
_malloc_r
__malloc_sbrk_start
_sbrk
_sbrk_r
__sf_fake_stderr
__sf_fake_stdin
__sf_fake_stdout


Tokens in 2, not in 1
atexit
_global_impure_ptr
__libc_fini_array
__register_exitproc
register_fini
sleepmgr_sleep
system_set_sleepmode
system_sleep
```
  

* AlarmMsg is an app symbol, it should definitely be included
* errno ... this is bare metal, no syscalls why is it there
* heap and malloc ... not used, why is it there
* fake stdout ... wtf?
* sleep stuff... should be in both... 
  
Oddly, #1 with the heap/malloc symbols and the 
AlarmMsg symbol had less .data usage.  
Pretty sure the 3 sleep functions are not using a lot of .data.  
  
# Get the size of a symbol
Turns out `nm -S my.elf` prints the size of each symbol.  
Since I only have a few to look at I can just manaully search.  
Quick reference for section codes:  
* "b" : BSS
* "d" : Initialized Data
* "r" : Read only data
* "t" : Text
  
Hmm, all of the #2 symbols are in the text section except the 4 byte `_global_impure_ptr**.  So much for finding the extra bytes in the .data section.
  
**Another Idea:  Scan through the .data section with `objdump`**  
The original ELF had 300 bytes .data and the new ELF has 1200.  I should be able to find the difference.  
Remember .data is __initialized data__ , and .bss is _uninitializedata__ , where most RAM variables are.
  
Ahh it appears the size difference is largely in the `.relocate` section.  Hmm, I believe that refers to the relocating of executable code from flash to RAM, so the code runs faster.  This could be either a bootloader section (which allows the bootloader to reflash the entire memory) , or some time critical routines eg. ISRs.  
Turns out not.  .relocate was actually RAM variables, not code.
Looking through it I saw a ~1k symbol called `impure_data` , which I googled to find out can be reduced by using a different stdlib, newlib-nano.  Checking the flags on the link step, I see I missed some!  I was only checking flags on the compile step.  Let's do the diff at the link step.  
  
OK, adding that flag `--specs=nano.specs` caused the data section to be just about right, off by 4 bytes.  BSS off by 28, text by 166.  Getting close.
