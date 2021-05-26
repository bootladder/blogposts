---
layout: post
title: "Product Variants.  Techniques.  What works?"
date: 2021-05-23
summary: "Product variants are everywhere.  How do you produce them fast and confident?"
categories:
  - "C/C++"
  - "Embedded"
---

## There are many ways to do it
  
I have created many different variants of many different products.  
What is a variant?  The definition is not strict here but we are talking about the Firmware specifically being a variant.
The difference between 2 FW images could be as simple as a single bit flag being a different value.  
Or it could be a completely different application running a common low layer stack.
Or a completely different stack running a common application.
Or a test program, or a debug program.  
Perhaps it is literally the same program being compiled with different compilers or a different architecture or CPU target.  
  
> Perhaps there is a need to combinatorically support all of the above variants, resulting in 10's or 100's of variants.  
  
  
What are all the possible ways you can write and generate firmware variants?  
They range from terrible ideas to usable ideas and there is no single answer and often it is a composition of many ideas.
  
The stupidest thing you could do is just change source code and rebuild.  This is probably the only thing
you can think of before you learn to use the tools that allow you to do things other than this.  
So I don't even consider this as an option.  
  
### Here are some techniques that can be used to create product variants

* preprocessor #ifdef's
* runtime if-else's
* runtime-initialization polymorphism eg. function pointers
* git branches
* common code + variant specific repositories
* 2 complete programs inside 1 executable, using main() to decide which one to run.
* Build system configuration

### The most important one is Build System Configuration

All of these have problems ie. tradeoffs except the last one, build system configuration.  
I believe that all variants should be generated at build time, ie. if you don't request a specific
build target, then every possible build target will be built when you run make or whatever.

> Think of the build system itself being a map from a git commit SHA to a set of binaries.

Some build variants can be expressed entirely by the build system, such as a -g flag for a debug target.
But not only that, all build variants(as stated above) **should** be expressed in the build system,
so this becomes the trivial case or base case for implementing product variants.  
  

Other types of variants can only be supported in the source code itself.  
These techniques can achieve the result ONLY by using the build system plus that technique.  
I think anyone would agree adding zero or less source is better than adding more source.  
So do as much as you can in the build system.  
  
### git branches
Somehow this seems like an intuitive idea to people.  It is but it isn't.  
Sure, you can think of a variant as being master branch plus the diff between master and the variant.  
But when master goes ahead, the variant is behind.  
Are you going to rebase the variant every time master gets committed?  
Or only when you need to release a new version of that variant and then realize that there are significant conflicts
and large bug surface.  

I personally don't like this idea but I am welcome to ideas supporting it.  
I like tags and merged branches but I don't like dangling branches.  
  
### common + variant repos
This also seems intuitive, perhaps because it is almost the exact same idea as above.  I don't have a problem with this in theory but in practice it doesn't seem to hold up.  
The line between what is common and variant specific is too unstable.  
And as with above you have more places of maintenance and thus more possible breakage to worry about.  
  

### 2 complete programs inside 1 executable
This idea sounds ridiculous but I actually like it conceptually.  
It just will very rarely be viable on an embedded system becausse of limited RAM and flash.  
The idea is you just add more code, don't change any code.  
Then at the very top level, as close to main as possible, you decide which "sub-main" to run.  


### runtime initialization eg. function pointers or control flow flags
This is similar to the above in the sense that at the top of main you select control flow.  
But the control flow conditionals are inserted into the existing code.  
For example, when sending a message, you might have a conditional to say "UART_SendMessage" or "Socket_SendMessage"
depending on some flag that was set at the top of main().  

This idea I believe is the best, if it is possible to achieve it.  
This is due to the fact that everything you care about in defining the variant is at initialization time.  
This exposes to the reader, the complete diff between base and variant, in terms of a set of parameters.  
There may be conditional logic deep down but it all viewable from the top.  


# There are 2 types of variants
### 2 modes 1 binary, or 2 binaries

2 binaries can be easier but then you have to maintain both.  It may not be easy
to switch 1 for the other.
If the variants are very different, then 2 binaries makes sense.  
But if it's more like 2 modes that can be run on a given piece of hardware,
it may be easier for everyone else (eg. customer) to have 1 binary,
and have the runtime mode be controllable either by you or them or both.

  
### What worked for me in a significantly varied variant, 2 modes 1 binary

Here is the process I used:  
  
1.  Checkout a new branch off master
1.  Develop the variant as if master never existed or will be scrapped
1.  Finish developing the variant
1.  Look at the diff between variant and master
1.  Decide where to insert conditionals
1.  Merge master up to variant
1.  Add the conditionals either as part of merge conflicts or after
