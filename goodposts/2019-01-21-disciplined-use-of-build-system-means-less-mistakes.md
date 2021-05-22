---
layout: post
title: Disciplined Use of Build System Means Less Mistakes
date: 2019-01-21
summary: How sophisticated is your build system?  Some ideas to compare.
categories:
    - Embedded
    - C/C++
---
# Disciplined Use of Build System Means Less Mistakes
The following scenario is a problem and it needs to be eliminated:  
  
1. I work on my code changes, generate a build, execute it, repeat.  
At some point I decide the code is good so I commit and increment the verison number.  
1. I deliver the binary with documentation.  
The client then decides, based on the documentation, that this is not what they want.  They clarify and I get back to work.  
1. I fix the code but I don't increment the version number because it's too small of a change, and the client would see 2 revs instead of 1.  
1. I re-deliver the code.  (note the delivery is a emailed zip file, and the binary file is a Intel Hex).  
Client tests the code and finds a definite mistake and a possible mistake.  The definite mistake I had thought I removed, but somehow slipped into one or both of the releases I sent out.  The possible mistake discovered in testing was due to misunderstanding of the documentation leading to blaming me, but it is most likely an environment or hardware issue.  
  
## Now let's identify the problems

**#1**, It's totally arbitrary when I decide the code is good and when I increment the version number.  
**#2**, The first delivery is pointless when the client rejects it before testing.  
**#3**, I didn't want to rev the version because there are not enough digits in the version number.  
**#4**, I delievered 2 zip files that are similar enough to be confusing.  The binaries have the exact same filename.  Now I don't have proof whether I made a mistake or they used the binary from the wrong delivery.  Moreover, since I only delivered a Hex and not an ELF, it's way harder to do the binary anaysis.
  
## Really the main problem here is:  Every time some wrong behavior happens, whether it's due to user error or environmental factors, it looks like my fault, and the burden of proof is on me.
  
# The solution is 2 fold:  Jenkins, and a better build system.
   
Jenkins eliminates email deliveries.  I don't deliver the binaries to the client, I publish the binaries with Jenkins.  This way, there is always a latest release in 1 place.  Never look back in an email or your own filesystem again.  Always go to the publisher.
  
The better build system includes tighter integration with source control and incrementing version numbers.  Definitely include some optional versioning like a build number or commit SHA, so the filename of the binary is always changing.  
That mistake I made, pointed at in #4 above, was due to myself putting test code in the production build.  Not unit tests, but a POST hardware test.  What the client saw indicated that I forgot to remove the POST.  I definitely remembered to do it, but somehow a mistake slipped in.  
I could have avoided this by parameterizing the build and generating extra build outputs.  There's the real final release, and then other parameterized builds like "myapp-0.0.0+WITH_POST".  This way, it's impossible to make that mistake.  No switching back and forth, just generate both and select one.
  
Combining the 2, this should guarantee that every build on Jenkins is unique and reproducible.
