---
layout: post
title: "Automatic Firmware Upgrades.  Stop saying OTA."
date: 2021-06-01
summary: "OTA is only the last mile, if anything"
categories:
  - "C/C++"
  - "Embedded"
  - "Testing"
  - "Ops"
---

In this post I will discuss the following topics:  

* Why is OTA a misued word.  Firmware upgrade is fine.
* What are the major components of a FW upgrade system?
* What are the challenges and risks in developing and/or using a FW upgrade system?
* How can you mitigate risk and make compromises?

### First, are you Agile?  Do you even have a Firmware Upgrade System?

If you're confident or ignorant enough to call your firmware development shop Agile, you'd better
have a firwmare upgrade system.  Even better, you should have a **fully automated** system.  
If you don't, how can you do sprints, demos, thin slices of end-to-end functionality, CI/CD, 
automated regression tests, etc?  
You can't!  
  
There's a big problem here.  

> The thing you need first is the hardest thing to develop.  Automatic Firmware Upgrade System.
  
Now, feel better and reassured, the reason you **don't** have a FW upgrade system is because it's hard,
you don't know what it is at a high level, and you don't have anyone who can implement it.  
It's OK to accept reality!  
  
### OTA is a marketing buzzword and only a small component

You have a device that is remotely located, and wireless.  It needs to be able to upgrade itself with a new firmware image.
The image needs to come from somewhere, wirelessly of course.  This is the OTA part.  **We're done right?  Wrong!**  
* The image needs to be identified as the right one for the target, 
* The target needs to be identified as the destination of the image transfer.
    This is generally the Internet going to an Internet connected device.  ie.  "Cloud to IoT Gateway".
* This leg needs validation as well as the OTA part.  
* There may be a need to update in batches or selective groups.
* There is a need to query and track what device has what version.
* There may be multiple sub-targets inside of a target that separately need to be upgraded.
* A user interface, a administrative interface, or both may be required.
* You may have to schedule system downtime during updates.
* You may have to schedule the upgrades with long times in between or staggered, or 
  all at the same time.  
* If introducing breaking changes, you will have to do a hail mary.  Good luck!

I will mention the S word.  Security.  I suggest considering security as a feature,
ie. feature of the finished deliverable product.  As opposed to being a piece of the communication
protocol and a requirement from the beginning of the project.  
In other words, you don't need security to do CI/CD, agile development.  But you do need security when you ship.  
  
I think it should be clear that "OTA updates" is not something you can just "have".  
The actual "Over the Air" piece is just one of many, and is of equal criticality.  

> The unfortunate reality is that there is quite a bit of product specifics in the whole pipeline,
> ie. not much is reusable across products.  


### Why is it hard to develop an Automatic Firmware Upgrade System?
#### Complexity and Risk

These are actually the same thing here.  The complexity results from the risk.  
A happy-path-only firmware upgrade is not hard to implement.
But, failures happen, which can brick the target.  This is the risk.
To gracefully handle failures, a lot more complexity is introduced,
which adds more points of failure, and thus more risk.  
  

### How can you avoid risk?  Compromise?

Here are some ideas.  
* Don't introduce breaking changes.  Leave plenty of reserved space in your protocols to allow extension.
* Configuration over code.  It is much easier and safer to update configuration than code.
* Write less code.  Less code means less bugs and less reason to update.  This sounds obvious but I'm totally serious.
* Make changes farther from the edge.  Try making changes at the gateway.  Treate endpoint updates as last resort.
* Start early.  It doesn't matter if "OTA" is a feature of your product.  You should have it anyway for development.
* Re-use the most critical components, such as bootloader, communication interface and protocol.
    This way you have the most testing around points of failure so you are more comfortable doing it "in the dark".  
    
    

### How can you achieve full automation?

You want full automation ASAP, for development and for production field updates.
The develop-test cycle is very related to the develop-release and the develop-field-update cycle.
So you might as well make them as close together ie. reusable as possible.  
  
  
In order to do this you need some stuff, probably out of scope for your Firmware Engineer, eg. UI widgets, backend code.  

* Storage and indexing of images:  web server with consistent URL and file naming convention
* Tool removal.  You can't use JTAG adapters because they're too expensive.  It's more like 1 per developer, not 1 per target.
  Remove the tool and solve the resulting problems.
* Automation pipeline.  Jenkins or whatever.  Reduce friction in the long cycles.


>  By long cycle I mean `git push` -> `build on CI` -> `store` -> `load` -> `flash target`
>  The short cycle is `build on IDE` -> `flash target`.  
>  You want more long cycles.  Short cycles are so fast you can't eliminate them but
>  you have to remove friction in the long cycles so they happen as much as possible.  
  

If you're at the point where you can trigger automated regression tests with a git push, 
you've made it pretty far and re-wrapping that into a "OTA" firwmare update system for use in production
field updates should be much more straight forward than doing it all from scratch.


## Conclusion:  You can do it, you should do it, but it is not easy.

* Start as early as possible
* Assume you will have an "OTA" feature even if you don't
* Integrate your "OTA" into your development and CI process
* Run automated regression tests so you have a reason to use your "OTA"
* The more you use your "OTA" the more testing you are doing on it
* Get multiple people involved, don't expect a single Firmware Engineer to take care of everything
* The cloud-to-gateway part is as important as the "over the air" part
* Try to do more flashing via your "OTA" and CI/CD, and less using JTAG

  
# Best of luck!

### Feel free to contact me to discuss :)

