---
layout: post
title: Jenkins Job DSL Part 2
---
The first article described using JobDSL to generate Jobs which were focused around deployment.  
The Job encapsulated both the building of the firmware and the prep for deployment.  
The prep involved this line:  
`echo $pathtonewelf > target/ffnbeagle-woodland-atmelice-J41800075898`  
Which associates a firmware ELF file to a deployment target.  
  
It worked but it fails now that I'm trying to build a single project for multiple targets.  
<https://bitbucket.org/bootladder/blinky-freertos-portable/src/master/>  
That single repository has everything to build for multiple targets.  
Running `./build.sh` generates the ELFs for the different targets.  
Point is, there is more than 1 ELF being generated.  
  
I think now what I have to do is, separate this in to 2 steps:  building and deploying.  

  
What's happening now is, the `blinky-freertos-portable` project gets built,
and multiple ELFs are generated in the same place at the same time.  
At this point, deployment can be triggered.  
  
Now, the Beaglebone in my lab has to grab the ELFs, 
decide which ELFs are loaded to which targets,
then do the loads.

Currently have a Kludged solution so let's rethink it.  
Currently, the Beaglebone has a polling loop.  
* Polls the Artifact Server for the name of the ELF to be loaded
* If the ELF already is on disk, stop here
* If not, wget the ELF from the Artifact Server
  * Then drive the flashing tool to flash the target
  
Polling via HTTP is undesirable.  But if polling can be replaced
by a trigger, the rest of it seems OK.  
  
Good thing is that the configuration to map (ELF -> Target) is
on the Server, not Beaglebone.  One less place to configure stuff.  
  
Now, last missing piece, auto edit configuration after a git push.  
Hmm, I could Kludge it and stick it all in the same job I guess.  
But details will be hidden, undesirably.  
Specifically, the details of the targets will be inside the file.  
  
Pipeline sounds good but let's just Kludge it for now.

  
# Refresher... , naming ...
Source Code is Pushed --> Bitbucket  
Bitbucket Triggers Jenkins  
Jenkins Builds For Targets (boards, CPUs)  
Jenkins Publishes the Builds, Associates build names with deploy targets.  
The word Target is confusing here.  There's a build target which targets a CPU and a board, but also a deploy target which targets a tool that the build target is sent to.  
  
So, at some point, there is a mapping from deploy target to build target.
A deploy target represents 1 deploy tool, and 1 build artifact.  
A deploy tool is only connected to 1 physical device.  
A deploy tool can deploy different build artifacts to a physical device.  
  
Note:  Confusion has occurred because for example, STM32F4Discovery is the build target, deploy target, and physical device.  What if I had 2 of these boards?  They would need to be identified by serial number.  That is, the Deploy Target is identified by serial number.  The build target can be named simply STM32F4Discovery.
  
Conclusion:  The JobDSL script does all of the following:
* Specify the source code repository
* Build for all targets
* Map deploy target to build target
  
# Ah crap, another complication.  AtmelICE can flash M0+ and M3 devices.  OpenOCD has to know which.
It's easy to create another openocd.cfg , so there'd be 2 for example:  
`atmelice-J41800075898-SAMDxx`  and `atmelice-J41800075898-SAM3XX`  
But in the fetch-and-load script that runs on Beaglebone, it needs to know if the physical device is a SAMD or a SAM3.  
**SOLUTION:** Have 2 fetch-and-load scripts, but only use 1.  
Because, at any given time, the Deploy Tool can only be connected to 1 Physical Device.  
Swapping a physical device means you swap the fetch-and-load that is running

