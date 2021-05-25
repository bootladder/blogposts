# Product Variants.  Techniques.  What works?
  
I have created many different variants of many different products.  
What is a variant?  The definition is not strict here but we are talking about the Firmware specifically being a variant.  
The difference between 2 FW images could be as simple as a single bit flag being a different value.  
Or it could be a completely different application running a common low layer stack.  
Or a completely different stack running a common application.  
Or a test program, or a debug program.  
Perhaps it is literally the same program being compiled with different compilers or a different architecture or CPU target.  
  
Perhaps there is a need to combinatorically support all of the above variants, resulting in 10's or 100's of variants.  
  
  
What are all the possible ways you can write and generate firmware variants?  
They range from terrible ideas to usable ideas and there is no single answer and often it is a composition of many ideas.
  
The stupidest thing you could do is just change source code and rebuild.  This is probably the only thing
you can think of before you learn to use the tools that allow you to do things other than this.  
So I don't even consider this as an option.  
  
* preprocessor #ifdef's
* runtime if-else's
* runtime-initialization polymorphism eg. function pointers
* git branches
* common code + variant specific repositories
* 2 complete programs inside 1 executable, using main() to decide which one to run.
* Build system configuration


All of these have problems ie. tradeoffs except the last one, build system configuration.  
I believe that all variants should be generated at build time, ie. if you don't request a specific
build target, then every possible build target is built when you run make or whatever.
Think of a mapping of git commit SHA to a set of binaries.

Some build variants can be expressed entirely by the build system, such as a -g flag for a debug target.  
But all build variants(as stated above) should be expressed in the build system,
so this becomes the trivial case or base case for implementing product variants.  
Other types of variants can only be supported in the source code itself.  
These techniques can achieve the result ONLY by using the build system plus that technique.  
I think anyone would agree adding zero or less source is better than adding more source.  
So do as much as you can in the build system.  
  
# git branches
Somehow this seems like an intuitive idea to people.  It is but it isn't.  
Sure, you can think of a variant as being master branch plus the diff between master and the variant.  
But when master goes ahead, the variant is behind.  
Are you going to rebase the variant every time master gets committed?  
Or only when you need to release a new version of that variant and then realize that there are significant conflicts
and large bug surface.  

I personally don't like this idea but I am welcome to ideas supporting it.  
I like tags and merged branches but I don't like dangling branches.  
  
# common + variant repos
This also seems intuitive, perhaps because it is almost the exact same idea as above.  I don't have a problem with this in theory but in practice it doesn't seem to hold up.  
The line between what is common and variant specific is too unstable.  
And as with above you have more places of maintenance and thus more possible breakage to worry about.  
  

# 2 complete programs inside 1 executable
This idea sounds ridiculous but I actually like it conceptually.  
It just will very rarely be viable on an embedded system becausse of limited RAM and flash.

# runtime initialization eg. function pointers or control flow flags
This idea I believe is the best, if it is possible to achieve it.  
This is due to the fact that everything you care about in defining the variant is at initialization time.  
This exposes to the reader, the complete diff between base and variant, in terms of a set of parameters.  
There may be conditional logic deep down but it all viewable from the top.  

