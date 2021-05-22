---
layout: post
title: Inject Mocks into your Cucumber Step Definitions
date: 2020-03-23
summary: Yo dawg, I heard you like injecting mocks.
categories:
    - Testing
    - Ops
---
# Use mockito and picocontainer to mock stuff like network interfaces
  
<img src="/assets/morpheus-testing.jpg" width="300" >
  
It's always a good idea to be able to run your code without "real" interfaces, whether it is a network, serial port, API, etc.  One huge reason is so you can test your code.
  
Why would you want to do that for your Cucumber tests?  They are the tests, not the code under test!  You wouldn't test your tests, right?
  
Well, have you ever had a step definition that did the wrong thing, had a bug, or had bad error handling/logging, and you didn't find out until you ran your tests and looked at the report?  In other words, you ran your test suite to test your tests, and your test had the bug, not the CUT?  This post addresses the palm on your face.

Actually, Cucumber tests aren't tests.  They are production code, all the way up to the features.  The "don't test your tests" idea matches up with "don't test your features".  The step definitions and everything below them should be tested.
  
What works for me is to keep the Step Definitions as thin and dumb as possible.  They are basically a wrapper/adapter to a library, which is SOLIDly TDD'd.  
  
But even with the above architecture, the Step Definitions still have to exist and there is stuff that can go wrong.  
For example:  A Step Definition does something and fails.  The test suite fails.  Oh wow, you may have found a bug!!  But the log wasn't there... oh no, it's a giant stack trace and none of it refers to what went wrong.  You have no proof that you caught a bug.  It's probably your code.  Now you have to explain why the tests fail.  
  
What if you could "mock run" your test suite, and see in your report the expected failure behavior?  What if this "mock run" executed in milliseconds to seconds, not minutes to hours?  
   
Well how do you do that?  Don't you need a running web app to run your selenium?  Don't you need to be connected to your network to access that thing?  
  
And here it is: ** Inject Mocks into your Step Definitions. ** 

<img src="/assets/xibit-testing.jpg"  width="300" >
  
Well you're probably using dependency injection already because if you aren't, you'd be using GLOBAL VARIBLES !!!   And because of this, it follows naturally to create some mocks and tell your DI tool to inject the mocks.  
  
  
Another benefit that I got from this, which is the reason I did it, is it allowed me to run (test) my tests while away from the office.  This in turn allowed me to develop more step definitions and receive more feature files from colleagues, and be basically 100% confident that when the time comes, the tests will run and behave exactly as intended.



  

