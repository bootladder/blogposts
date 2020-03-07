---
layout: post
title: Improving server logging and admin
---
Sometimes my blog update pipeline fails.  Could be for many reasons:  
* Travis missed a webhook
* freejekyllbuilder.com down or failed
* SSL cert on deployer expired
* deployer can't deploy
  * Bad file permissions
  * Bad system state
  
Finding the problem basically means tracing
the flow in order of execution until the
break is found.  
  
I'd much rather just be told where/what the break was and just go to immediately fixing it.  
  
3 things must be done:  
# Identify the error 
eg. check error code, check system state with more code, match an error message  
# Handle/log the error
Stop pipeline, retry, try to fix the break, log.  
Could be restarting a daemon, or retying a step "inside a daemon".  
Obvious trick: `set -e` in a bash script.  
  
Need a central log.  Can't look at each service/pipeline step for logs.
Perhaps slack?  Text Message?  
  
Central status server?  
  
Need to seprarate logs from freejekyllbuilder.com from httpsdeployer.  
Users should have freejekyllbuilder.com/status endpoint.  
httpsdeployer needs log of triggers.


# Remove extraneous logs
Obvious trick: `mycommand > /dev/null`


# Should I switch from bash to a "real language" ?
Lose a lot of shortcuts offered by bash.  
Definitely should be an interpreted language eg. python.  
Don't want to bother compiling and deploying binaries.
