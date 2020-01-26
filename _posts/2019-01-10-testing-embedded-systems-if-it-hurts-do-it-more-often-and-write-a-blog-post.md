---
layout: post
title: 'Testing Embedded Systems:  If it hurts, do it more often.  (and write a blog
  post)'
---
# "If it hurts, do it more often" -- XP

This quote generally applies to CI/CD.  
But you know what?  CI/CD is a lot easier on embedded systems than for web/mobile apps.  
Continuous Deployment is hardly applicable because... it means continuous OTA updates.  
CI/CD means you can always deliver...  Deliver what?  A firmware image to your Asian CM?  
  
Honestly I would love to have an Asian QA department testing my embedded systems before they get to the CM.  But given I don't speak Mandarin, there's about a 0% chance of it being successful, even at a $0 price tag.  
There are some very good testing houses in the US, but they're too expensive because their clients are government contractors.  They're generally more for regulatory testing anyway, equipped with extremely expensive equipment for regulatory testing.  Not for feature acceptance and regression testing.  There isn't a testing house specializing in "poor man's RF testing".
  
Anyway, I'm responsible for testing my firmware on real hardware before it gets to the CM.  If there's a bug, it's going to show up in the field many weeks later, and though the code can be fixed immediately, the change won't get deployed for another many weeks.
  
Setting up manual tests, dealing with physical connections, RF signals, batteries, damaged hardware, etc. is the annoying part about embedded systems.  This is the crap that takes all day and you're still blocked.  The part that was supposed to take a day but took a week.  
  
## If it hurts, do it more often
The idea is that if you force yourself to do it more often, you'll figure out how to automate it.  When you automate it, it won't hurt anymore.  
  
Let's add a Murphy-esque corollary:  
## If you have to do it once, you're going to have to do it again
Similar to manual deployment of web apps, manual testing of embedded systems will be repeated ad nauseam until it is automated.  
  
  
# OK now let's analyze:
**Major Questions:**
1. **What hurts?  What are the pain points?**
2. **How do we automate the pain away?**

## 1.What hurts?
* Hooking up Debuggers, Serial ports
* Keeping track of multiple devices
* Repeated sequences of Bash operations
* Intermittent hardware performance/behavior characteristics

## 2. How do we automate the pain away?
More documentation, brah.