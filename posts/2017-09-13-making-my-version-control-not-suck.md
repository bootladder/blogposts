---
layout: post
title: Making my version control not suck
---
I develop and maintain firmware for a family of products.
There are 5 different firmware projects here.
They're all releated in communication protocol, as well as other things
such as beep frequency, LED blink patterns etc.

But now I'm faced with splitting all 5 of those into 3 versions.
So now there are technically 15 different firmware images that need to be
available for download and documented to identify what they are.
  
My (git) version control system until now was obvious.  Master goes forward, occasional branching and merging, tags on version releases.
  
It would be possible to use branches and tags to rebuild any version of any project.  But here's the complication:  there is a in-house shared library used by all 15 firmware versions.  That library also has 3 versions.  In other words the correct version of the shared library has to be available when building a version of firmware.
  In more other words, to build a version of firmware, the project repo has to be checked out at the right version, as well as the shared library checked out at the right version.  Finally, the build config (eg. Makefile) has to identify and locate the correct shared library to be linked.
  
I like stuff like Ruby, Rust, etc. where you have a config file (yaml) to specify dependencies.  
  
I think I can improve my Makefile to do what I want.  
It would be as simple as git checkout my_commit followed by make.
The Makefile at this commit should build the proper version.  The Makefile itself identifies its own version and the dependency versions it needs.  It can then git clone the dependencies using git submodules.
  
This way, a complete build with all dependencies lives inside 1 directory.  If a submodule is updated or branched, it won't affect previous builds.
  
# Strategy:  use a fresh machine (VPS) to recreate the build
  
I don't want to be confused with currently existing directory structure.  Building the same firmware version on a different machine is a good way to shake out things like hard coded paths breaking.  
Ideally all builds would happen on this separate machine.  I guess that's part of Continous Integration?
  
# --- Makefile needs source located in a hard coded directory
  
Project needs a library or some source somewhere else to compile.
The Makefile can find the source with a VPATH and find the headers
with -I compiler flags.  But, that enforces a certain directory structure.

# --- New rule discovery?: Only create directory structure for sub-directories.  Never depend on paths starting wtih ../ or ../../ etc.
  
This rule allows you to create directory structure, but doesn't cause problems because the structure is all below current directory.  
Basically, enforced sub-directory structure is coupled entirely to
the populating of that sub-directory and makes no assumptions about directories above current directory.  
In other words, if you have enforced structure, the build system has to automatically conform to that structure.  Let the Makefile create the directories with their correct names, populate them with files.
  
# --- VPATH Issue with git clone
  
I have a repo prereq inside my all target.
The purpose is to clone the dependencies,
so Make can then refer to those dependencies and build the target.

```
always: 
repos: always
        @echo hello
        if [ ! -d "../../mysrcrepo" ] ; then \
            git clone https://mysrcrepo ;\
        fi 
        if [ ! -d "../../mylibrepo" ] ; then \
            git clone https://mylibrepo ;\
            make; \

        fi 
```
  
It almost worked.
  In the case of a cloned library, the make command is run, which builds the library.  That worked.
  But in the case of cloned source, the VPATH didn't catch the source files that were cloned.  When running make a second time, it would work.
  
Well apparently 2 things are wrong with that, as I've read.  
1, VPATH doesn't work on generated files.  Not sure what that means specifically but the way I made sense of it is this:  You can add directories to VPATH but if they don't exist when make is run, the VPATH won't see that directory.  Even if the file exists at the time it is needed, it won't be found.  That explains why the build works the second time. 
  
2.  Recursive Make Considered Harmful?

# Discovery:  how to do git inside make
  
https://stackoverflow.com/questions/15602059/git-shortcut-to-pull-with-clone-if-no-local-there-yet
  
Using this I can solve the problem in the above snippet
        if [ ! -d "../../mysrcrepo" ] ; then \
  
If the directory does exist, nothing happens.  What I want is a git pull in that case.
  

