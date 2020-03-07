---
layout: post
title: 'Code Review: Embedded C'
---
I learned a lot at this last job I had.  So much,
I realized the code from my last job sucks.  
Now I want to revisit it but it's going to be a drag.  
# why?
  
Multiple reasons
* Smelly code, ie. comments, long functions
* Not enough refactoring
* Didn't take the time to write clean, SOLID code.
* It's just not good enough
  
A big thing was:  **no tests** .  
* Without unit tests you lose a layer of documentation
* That doc layer gives you the chance to state the intended behavior
  
Changing the code is risky in the following ways:  
**Ordering, sequence** of function calls ie. X is called before Y.  
**Standalone call must be made** maybe for a side effect.  
This particularly goes for hardware init code.  
  
Another thing:  **messy build system**   
I had experimented with make, CMake, different testing frameworks, g++, etc.  
I never decided on one and now there's scattered fragments of all the experiments.  
Moreover, there's crap from the very beginning of the project when I used
vendor libs and let the vendor IDE handle my build.
Basically the following happened:  There were unused HW driver source files
that all depended on each other, so removing stuff would break the build,
requiring more stuff to be removed.  This is why you don't let crap stay in your source tree.  
One unfortunate side effect of cleaning the cruft is the build size may change,
so you don't know 100% that you built the same code.
  
I've found that C build systems are the most verbose and 
are a more important piece of the project than for build systems for other languages.  
Mostly because the C tools aren't as high level.  
  
So it's really important to get it right.  Not just barely usable.
  
Another thing:  **duplication ie. non-DRY**
  
I think DRY relates to project file structure.  
I have 5 product variants.  So I had 5 projects, 
ie. 5 git repos, ie.5 independent source trees, 5 build systems.
  
I have 5 copies of hardware init code.  But this made sense
because they're in general slightly different.  
  
I spent a lot of time thinking about this problem but didn't get
a clean solution.  Now I'm thinking it's better to have everything
under 1 project, 1 git repo, 1 source tree, 1 build system.
  
Before, with 5 projects and a ton of common code, what I did was
have a common repo which would be cloned by each project and
used in the build.  
Now, instead of 1 repo using a common repo, it'll be 5 projects
in 1 repo sharing common code in a directory in the same repo.  

But 1 downside to the monolith is it's not as modular.  
  
1 more thing:  **Branches per sub-variant**  
There's 5 product variants and 3 sub-variants per.  (making up a word here).  
It was 1 repo per variant and 1 branch per sub-variant.  
I had been keeping the sub-variants in a tight loop so as
features were developed it was a simple rebase of the other branches.  
  
Basically the way this is going is to **Continuous Integration**,
which means 1 branch, master, for _everything_.  

  
  
# Rehab
* Clean up the source tree.  No vestigial crap.
* Clean up the makefile.  No unused defs, ghost targets.  No stupidly long lines
* Start consistently building on Jenkins.
The more build history you have, the easier it is to
catch when a bug gets in.
If you clean/refactor and expect the final binary to stay the same size, track it and assert it.

# How to merge multiple repos into 1
* Create a new repo with 1 dummy init commit  
`git init;touch init; git commit -am "init"`
* Add a remote, fetch and merge a branch  
`git remote add remote1 /path/to/repo`  
`git fetch remote1 master`  
`git merge remote1/master`  
`git branch merge-point-of-remote1`  
* At this point you may want to move everything under a single directory to avoid merge conflicts.
* Add a second remote, fetch it, checkout it, rebase
on the HEAD of the previous merge.  
`git remote add remote2 /path/to/repo`  
`git fetch remote2 master`  
`git checkout remote2/master`  
`git checkout -b local/remote2/master`  
`git rebase merge-point-of-remote1`  
* You'll notice that .gitignore conflicts every time a commit
modifies it.  That's OK just resolve the conflict each time
and `git add .; git rebase --continue`
  
**But I'm scared that I wont be able to revert back and use the old split-up repos.**
It would be too tedious to bring the code back into the old ones.  
Well if you had moved each repo into its own directory when merging them in, they would all be .. __orthogonal__ ?  So, you could
look at commits that affected files under one directory,
and you'd effectively get the same history as if it were a separate repo.  
Issues arise when you factor out common code into another directory, but you'll just have to include the common code when you "slice out".  
Another thing is due to rebasing, the commit SHAs are
different but you'll figure something out...
  
**But now Jenkins has to build all 5 variants**  
1.  You'll eventually have integration tests
2.  If you change common code you'll want to feel safe about the other 4 variants.  
  
    
# Really, why is the Makefile so long?
Let's go through and write down literally everything it does.  

Pass #1 , reading top to bottom
* Version Number
* Output file name
* Toolchain aliases (gcc objcopy etc)
* VPATH
* Object Names
* Linker flags
* Compiler flags
* Compiler Defines 
* Clone and build common dependencies
* Binary outputs (gcc objcopy size etc) 
* Same thing for test build
  
And, it's about 400 lines.

Pass #2, following flow of the all target
* Clone dependency repos and build
  * cd, clone, checkout, make
* Build each object
  * list of object names
  * compiler flags, defines 
* Link, binutils outputs
  * output names

Pass #3, looking for duplication and cruft top to bottom
* dupe  : toolchain def: `CROSS_C_COMPILER = arm-none-eabi-gcc`
* dupe  : vendor library VPATH
* cruft : lists of 100 object files and VPATH dirs
* dupe  : list of vendor library object files
* cruft : list of 100 -I include flags
* dupe  : compiler and linker flags
* dupe  : vendor compiler defines
* dupe  : vendor target (clone dependency repos)
* dupe  : targets for building object files, linking exe and binutils
  
Pass #4, looking for duplication and cruft 
following flow of the all target  
* dupe  : all target, repos target, output exe target, object target
* dupe  : compiler flags, defines, warnings
  
  
# Remarks  
Really the only specifics to each Makefile were:
1.  List of object files being built for that project and VPATH dirs,
and even that list had duplicated entries for vendor and common.  
  Moreover those lists should not need to be explicitly listed.
  There must be a way to implicitly ie auto find/generate the objects
  being built.
2.  Names, version number
