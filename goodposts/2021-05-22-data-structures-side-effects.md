---
layout: post
title: Data Structures and Side Effects
date: 2021-05-22
summary: Side effects ruin pure data structures.  Avoid them!
categories:
  - "C/C++"
  - "Embedded"
---

### How pure are your data structures?

When you write code for microcontrollers you tend to have the source code of the entire system.  
What I mean is, no shared libraries, no static libraries.  You can see the entire memory map,
every instruction and every initialized data.  You have a view into the system that no desktop or web programmer
could ever have.    
But, because of this, it is far too easy to insert code in places where they shouldn't go.  
One place where this happens is at initialization, but what I'm talking about here is data structures.  
  
We all learn about how data structures and scoping can be used to encapsulate code and data.  
But it is far too easy to break the rules.  Even worse, it's sometimes not easy to not break the rules.
ie. the path of least resistance is often wrong.  
For example you may forget that a side effect is a side effect.  
  
Here's an analogy:  

>  Would you modify and recompile the Linux kernel and reboot your entire system,
> for the sole purpose of supporting one of your custom applications?  
  
No of course, that's insane.  But that's the view we have of the system as embedded system people,
and I see the equivalent being done all the time.  

  
### Why are side effects bad?  
  
From the CPU and programmer's perspective, side effects are the same as global variables.
They make your code non-deterministic when reading,
and they make the outside world non-deterministic when writing.  

Code with side effects cannot be tested, for 2 reasons.
Obviously, you need the real world to interact with the code and sometimes you can't fake it.  
Moreover, code with side effects is not portable, because they are target specific.
Non portable code has far reaching bad consequences.
You can't test on the host because the code won't compile due to target specifics.
Even within the target you can't reuse the code anymore.
If you try to reuse it, you will then have 2 nearly identical copies of the same data structure, but each with different side effects.  
It sounds crazy on paper but I guarantee you people do it.  

### Why is it hard to avoid and why do people do it?
  
The reason why people put things in the wrong place is because they are confusing
control flow dependency with symbol dependency.  They are 2 different things, but as a novice you don't see their independence.  
This comes from the __Dependency Inversion Principle__.  
What happens is you start at `main()` and look down the call tree.  You know temporally (ie. in time)
where to add the code.  But you don't know spacially where to put it.  So you make a decision,
it has to be somewhere and none of the places seem right or they all seem wrong so you just stick it somewhere
because you know it'll work.  Maybe you get a compiler error that you don't want to fix or don't know how to fix,
and then you put it somewhere else that doesn't give  you an error.  
  
You intiuitively understand the control flow dependence on this new code.  Say it's a 1 line function call.
You know it has to be called, and you know where on that call tree it can be, and where it cannot be.  
If you don't deliberately invert edges in the the control flow dependency tree, then the symbol dependency tree will be the same as the control flow dependency tree.  
  
Here's a yardstick to see this in your own codebase:
At the top of your .C files, pay attention to how many header files are included.
This includes header files including other files.  
The more you have, the easier it is to do the wrong thing because less will stop you from doing it.  
The less you have, then you will have to add more includes to allow you to call out from farther places,
and thus you will think about it a bit.  
  
Pay attention to which level of abstraction these header file includes are at.  
Did you write them?  Are they lower level things like filesystem, network?
Are they target or platform specific?  Do you have a `#include <windows.h>` or `#include <mytarget.h>` ?  
  
> Are you doing target specific stuff in places that actually have zero relation to the target?
> Such as state machine implementations, data structures, business logic, parsing data from the network?  


### Why is it good to care?
  
I have on multiple occasions done significant modification of code.  
Not refactoring, because refactoring means the behavior doesn't change.
I'm talking about changing the behavior by mostly rearranging code and not much new code.  
When you do this, you are hacking pieces of the system and bolting them on elsewhere.  
If you deal with pure data structures that are tested, you never have problems except at the call site,
ie. wrong parameters.  You won't have any problems due to encapsulated code producing side effects.



