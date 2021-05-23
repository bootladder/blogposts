---
layout: post
title: Data Structures and Side Effects
date: 2021-05-22
summary: Side effects ruin pure data structures.  Avoid them!
categories:
  - "C/C++"
  - "Embedded"
---

# Sorry, this post is WIP, but thanks for visiting!


When you write code for microcontrollers you tend to have the source code of the entire system.  
What I mean is, no shared libraries, no static libraries.  You can see the entire memory map,
every instruction and every initialized data.  
  
Because of this, it is far too easy to insert code in places where they shouldn't go.  
One place where this happens is at initialization, but what I'm talking about here is data structures.  
  
We all learn about how data structures and scoping can be used to encapsulate code and data.  
But it is far too easy to break the rules.  
For example you may forget that a side effect is a side effect.  
  
# Why are side effects bad?  
  
From the CPU and programmer's perspective, side effects are the same as global variables.
They make your code non-deterministic when reading,
and they make the outside world non-deterministic when writing.  

Code with side effects cannot be tested, for 2 reasons.  
Obviously, you need the real world to interact with the code and sometimes you can't fake it.  
Moreover, code with side effects is not portable, because they are target specific.  
Non portable code has far reaching bad consequences.  
You can't test on the host because the code won't compile due to target specifics.  
Even within the target you can't reuse the code anymore.  
A data structure is no longer a data structure when side effects are added to it.  
You will then have 2 nearly identical copies of the same data structure, but each with different side effects.  
  
I have on multiple occasions done significant modification of code.  
Not refactoring, because refactoring means the behavior doesn't change.
I'm talking about changing the behavior by mostly rearranging code and not much new code.  
When you do this, you are hacking pieces of the system and bolting them on elsewhere.  
If you deal with pure data structures that are tested, you never have problems except at the call site,
ie. wrong parameters.  You won't have any problems due to encapsulated code producing side effects.



