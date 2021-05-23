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
2.  You lose the ability to share the 3rd party code.
3.  You change the behavior of pure data structures ie. side effects
4.  You are making changes that are hidden in the depths, instead of in view from the top
5.  You can break stuff and not know where, because the breakage is up the call chain.

One example of this is in the initialization of a communication stack.
Imagine a SPI peripheral such as an Ethernet tranceiver is being driven by a 3rd party stack.
The code you receive has an initialization routine, which you call, and it makes some reference application work.
Of course you have specific requirements so the init() they gave you doesn't apply.
Do you modify init() inside the source code of the 3rd party stack?

I bet you did, but should you have?  Violating the OCP here has consequences which you should be aware of.

What do you do when there is a product variant which is almost the same but has a different init() requirement
of the same Ethernet tranceiver?  There is no longer such a thing as init().  There are 2 inits().
Are you going to have init_A() and init_B() ?  This is only making things worse.

If you don't usse git you're literally screwed because now you can't tell what you've done.
But of course you use git.
Now what happens when you have a new product, it's not a product variant.  But it still uses the same tranceiver.
You have this frankenstein stack now, you can't use the stock stack.
But frankenstein only worked for your old products.  Now you can't use either the stock stack or the frankenstein stack.
Now you have 3 differnet versions of the stack and some of these stacks can be big, 10k's of lines.


So what's the answer here?
You create and maintain your own layer that sits on top of the stack.
Yes, a higher level can go all the way down to the lowest level.  It is not a requirement that
each layer be only accessible by the layer above it.
So this means you can call PHY_init() from the application code.
I personally have been experimenting with the pattern APP_PHY_Init().


So the result is 1 copy of the 3rd party code, you could even compile it into a library to enforce the fact
that it should never change and now it's just 1 file plus header files.
The 1 copy plus your layer.  The layer has lots of hooks into the stack but the layer itself is thin,
and it only contains things you are interested in.  It is the diff between your requirements and the underlying stack.
