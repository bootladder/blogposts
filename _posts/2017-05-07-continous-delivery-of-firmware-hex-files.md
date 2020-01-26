---
layout: post
title: continous-delivery-of-firmware-hex-files
categories: guide
---
  
I had a short, one-off project based on a code base in production.  
Writing the source was easy, but the deliverable was 20 hex files.
Each hex file contained the exact same firmware image, with a single piece of data in flash changed to some value.  
I had my Makefile set up so I could set that value inside the Makefile.  
But to build all 20 required editing the Makefile and running a clean build.  
I did that once and released to the customer.  
Obviously the customer changed the spec because they couldn't translate what they wanted into the spec correctly, and they discovered this by getting what they specified originally.  
  
So now I have to repeat the process except the change to the source code is now a single line.  The entirety of the time spent for this job is in waiting for 20 clean builds.  
  
I have a postulation here, needs to be reworded:  
  
As soon as the build system use case goes beyond a single command-line command, eg. "make", you have to automate your build system.  
  
In my case I need the system to do a clean build, 20 times, each with a different value for some variable in the Makefile.  
  
So I did this the fastest simplest way possible for me to implement.  
I went with the following:  
Make 20 different targets in the Makefile.  target1,target2,target3....target20.  
For each target, a target-specific variable can be specified, and in this case
it overwrites a variable from the top of the makefile which is used in the all target.  
So, each target simply sets that target-specific variable, then has a dependency on the clean and all targets.  So we get a clean build with the variable set.  This variable also conveniently appears in the name of the hex file so that takes care of getting 20 unique filenames.  
Then, to actually build all of those targets, I just made a shell script that does make -f MyMakefile targetN for all targets.  
Really a crappy solution but that's all I could do at the time and it still achieves automation.  Cool!  

