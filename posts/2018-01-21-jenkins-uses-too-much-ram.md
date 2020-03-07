---
layout: post
title: Jenkins Job Migration (ie. Jenkins uses too much RAM)
---
Dang, I finally figured out why my Docker containers were failing shortly after starting.  
That Error Code 137, Out of Memory.  
Really it's my stupid Redmine container that's taking up 30% of my 1GB RAM.  But I can't do anything about that.  
Jenkins I can consolidate the 3 instances into 1.  
Previously I had 1 Jenkins per toolchain, ie. 1 for arm-linux , 1 for arm-none, 1 for golang, etc.  I liked this because it keeps the toolchain installs independent.  
Now I have to use 1 Jenkins container with all the toolchains inside of it.  
  
First I'll have to port all the Jobs into the 1 Jenkins instance.  Then I'll see them in the Jenkins UI, so I can run builds which will fail due to no toolchain installed.  Then I can install the toolchains.
  
Great, copy pasting the jobs/ directories worked, I now see the jobs in the Jenkins homepage.  And of course, running the build fails due to non-existing toolchain.  
So let's open the Dockerfile and install some toolchains.  
  
Well it works, pretty obvious.  
  
Just remember folks, 1GB RAM is easily plenty and easily nothing.  Pay attention to how much your stupid high level frameworks and languages use.  
  
