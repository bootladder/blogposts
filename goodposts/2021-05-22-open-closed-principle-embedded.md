---
layout: post
title: Open Closed Principle (OCP) in Firmware
date: 2021-05-22
summary: You've heard of SOLID but can you explain why OCP is important
         and when it is being violated?
categories:
  - "C/C++"
  - "Embedded"
---

I believe all programmers should be familiar with SOLID principles, not as dogma but with a deep skeptical
pursuit of the truth.  They don't many any sense unless you really think about it hard.
You should be able to talk at length about each one.

The Open Closed Principle, OCP, has been coming up a lot in my firmware.
There are 2 major occurrences of violations in my own code.
One is in using 3rd party or vendor stacks such as communication stacks or HALs.
The other one is in data structures.

Check yourself:  If you add code to a 3rd party codebase, you are violating OCP.
If you add code to functions that are being called at some point, you are violating OCP.

Is it a strict rule to never violate OCP?  Of course not.
But why is it bad sometimes to violate it?

1.  You have effectively forked the 3rd party code.  Good luck maintaining it.
2.  You lose the ability to share the 3rd party code.  Are you sharing the author's copy or yours?
3.  If you change side-effect behavior that's really bad
4.  You are making changes that are hidden in the depths, instead of in view from the top
5.  You can break stuff and not know where, because the breakage is up the call chain.
6.  You break tests and then change the tests or delete them.  Waste of usable functionality.

One example of this is in the initialization of a communication stack.
Imagine a SPI peripheral such as an Ethernet tranceiver is being driven by a 3rd party stack.
The code you receive has an initialization routine, `init()`, and it makes some reference application work.
Of course you have specific requirements, so the `init()` they gave you doesn't apply.
Do you modify `init()` inside the source code of the 3rd party stack, so it matches your requirements?

I bet you did, but should you have?  Violating the OCP here has consequences which you should be aware of.  

What do you do when there is a product variant which is almost the same but has a different `init()` requirement
of the same Ethernet tranceiver?  There is no longer such a thing as `init()`.  There are 2 `init()`'s.
Are you going to have `init_A()` and `init_B()` ?  This is only further violating OCP making things worse.

If you don't use git you're literally screwed because now you can't tell what you've done.
But of course you use git.  
Now what happens when you have a new product, it's not a product variant.  But it still uses the same tranceiver.
You have this frankenstein stack now, you can't use the stock stack because you put too much in frankenstein.
But frankenstein only worked for your old products, it doesn't cover the requirements for the new product.
Now you can't use either the stock stack or the frankenstein stack.  
Now you have 3 different versions of the stack, they're all crap, and some of these stacks can be big, 10k's of lines.  


# So what's the answer here?  
The thing about C is that there is almost nothing the language can do to help you, because the
language does so little.  But that's what makes C so great to me; it puts all the abstraction in my own domain.  
So you think about the solution itself (extend the code without modifying it), not the implementation of the solution (inheritance, polymorphism).  
Moreover, neither inheritance or polymorphic solutions will work for this particular problem,
because of the `private` functions, which will be discussed now.  

Here's what worked for me.  
**You create and maintain your own layer that sits on top of the stack.**  
A higher layer can go all the way down to the lowest layer.  It is not a requirement that
each layer be only accessible by the layer above it.
So this means you can call `PHY_init()`, and the functions that it calls from the application code.  

I personally have been experimenting with the pattern APP_PHY_Init().  
1 option is to ignore the original ie. stock `PHY_Init()`, don't call it, and replace it with your own `APP_PHY_Init()`.
The definition of `APP_PHY_Init()` might be very similar to `PHY_Init()` but it is yours to maintain in your own codebase.
So we're in a sense trying to bypass the PHY layer and do all the calling of internal PHY layer functions from the application layer, but only for initializing.  
  
Sometimes this is not possible because `PHY_Init()` was doing stuff that is static, ie. private, ie. not accessible from other
translation units.  This means you can't do the above.  After all, isn't the whole point of having layers and private scope so that it limits coupling and maximizes cohesion? ie. literally to prevent me from doing the above?  

So, an option here is to add functions to the PHY layer, which are just wrappers around those private functions to allow a higher layer to call them.  DO NOT just change static to non-static or private to non-private.  You only add, you do not change.  This way you cannot alter existing behavior that is depended on by other code, ie. this is always a forward compatible change.  
Yes, you have some downsides such as the fact that you have effectively forked the library.  
However you do have some protection such as the fact that if you mismatch library versions ie.
try to use the stock stack instead of yours, the compiler can help you detect this via undefined symbol errors.  
The compiler says the new stuff you want is not there.  If you change behavior, the compiler cannot help you.
   
Back to the above statement about `private` functions preventing the solution.  l believe there is no solution.
Inheritance doesn't work because derived classes still can't see the private functions or data.  
Polymorphism ie. using an interface, is just an ugly hack duplicating the module in question.
ie. the solution is to have `PHY` and then `PHY_Better`, where `PHY_Better` is just a copy of `PHY` 
with a couple extra things to support your application.  

So in this scenario with the `private` stuff, I think you have to violate OCP and modify the code, or ditch that part of the code and rewrite your own.


# Recap
The ideal result is 1 original, unmodified copy of the 3rd party stack, plus your application adapter layer.  
The layer has lots of hooks into the stack but the layer itself is thin, 
and it only contains things you are interested in.  It is the diff between your requirements and the underlying stack.  

You could even compile the original 3rd party stack into a static library to enforce the fact
that it should never change and now it's just 1 file plus header files.  

If you can't do this, I think you have to modify the stack, but do this in a way that still keeps the OCP in mind.  
Know that you are violating it and there are consequences down the road.
