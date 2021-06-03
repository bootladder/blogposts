# Case Study: MQTT Functional BDD Testing Framework  

### The Client:  XYZ Technologies, Inc.

XYZ, formerly ZZZ, manufactures residential security and home automation systems.
They developed a communication device to connect legacy security panels to the Internet.
The legacy and retrofitting nature of the product meant that a lot of work was re-cycled from previous products,
which shortened the schedule dramatically.  
  
The client wanted maximum test coverage, only covering the areas that were new development.
Most importantly, they wanted the test coverage from the beginning, to go alongside with development.  
  
  
I quickly created a test automation framework and pipelining, got the development team on-board,
and within a few weeks the testing was lock-step with development.  The client got valuable feedback 
from the testing all the way to the end of the project.  

  
### Overview

Testing IoT products is difficult because the end-to-end paths are longer.
A typical test case involves hardware interaction, wireless communication,
cloud interfacing, backend and multiple user interfaces.
Because there are so many points of failure, engineering teams want a way to test their
subsystems without running into failures out of scope.  

Specifically on Embedded Linux systems, there is a greater challenge because within a single device,
the line separating end-to-end testing and unit testing is wide and technically advanced,
which makes traditional engineering and QA roles less clear.  

To solve these challenges and achieve excellent test coverage, a custom test automation framework
was created, and a test execution pipeline was integrated into the CI build pipeline.  
  
A unique style of Behavior Driven Development (BDD) testing was employed with the custom framework,
which allowed for high test coverage, ease of use, consistent execution and useful reporting.  



### The Challenge:  Do the important work, make it easy, don't repeat existing work

The developers had excellent coverage with unit tests so we did not want to double-cover any of that.
The part not being covered was the inter-process communication (IPC) between processes running on the Linux system.
The IPC mechanism being used happened to be MQTT but this idea applies to any other IPC mechanism.  

System, ie. end-to-end testing is normally delegated to QA testers.  But in this case of an Embedded Linux device,
system level is not end-to-end.  ie. there are a lot of technical implementation details, as opposed to business domain details.  However, developers aren't as concerned with the system level as they are with
the process level that they develop and unit test at.  

### The Solution:  Test at the boundaries, let the developers write the tests, do not get in the way
  
Thinking of a Linux system as a collection of inter-communicating processes, we chose to 
create a test solution to cover the IPC interfaces only.  This allowed the test framework itself to be
very thin, essentially only concerning sending, receiving, and validating messages across the IPC interface.  
The benefit was that it is easier to show the developers how to use it, and give them the confidence that
it is valuable to use.  This essentially removed the need for QA testers to be concerned at this level.  

As a framework author and maintainer, I want to let my users do as much as possible without having
to ask any questions, especially to ask me questions.  The less questions I receive, the better quality my work is.
So that meant the framework had to be light, thin, obvious.  
  
The result was a complete automation framework spanning all the way from the test case descriptions,
to the report being delievered after executing the tests.  
  
After I proved this implementation the team leader had all of the developers running tests using this system, 
and ops people replicating the pipelines to increase coverage.  
Once I trained an initial set of people, they trained each other.  


### Results:  Numbers don't lie

Test coverage was the critical result.  

* 523 Test Cases
* 5 Software Components under test
* System in use for 12 months and counting
* 1342 test executions and counting


Because the test suites would run after every git commit, it was very easy to catch bugs.
The number of bugs caught is hard to measure but anecdotally I was told that every 
developer had caught multiple bugs with this system, saving a huge amount of time and stress.  


Also there were many intangible results which are enjoyable for everyone  

* Teach developers about ops, testing
* Teach ops people about testing
* Write once, use forever
* Breaking the paradigm and still being successful


### Some rules are meant to be broken

BDD and Cucumber are generally not used in the way we did, but what we did was very successful.
Retrospectively I would not have done it any other way.  
  
  
So, trust your intuition so you can read more stuff and take in more opinions but always be solid.
  
  
### Credits and Thank you's

Thanks to the people who used the framework.
