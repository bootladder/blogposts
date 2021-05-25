# Case Study: MQTT Functional BDD Testing Framework  

As an employee of Resideo Technologies, Inc, I developed a test automation framework for testing an embedded Linux device.  
After leaving Resideo, I released the framework and continue to support it.  
  
We had a QA department supporting the Engineering department.  
QA did manual and end-to-end testing, Engineering did unit testing.  
  
But on a Linux system there is system level testing for the system itself as well as the greater system it is a part of.  

The developers had good coverage with unit tests so I did not want to double-cover any of that.  
The part not being covered was the inter-process communication between processes running on the Linux system.  
The IPC mechanism being used happened to be MQTT but this idea applies to any other IPC mechanism.  
  
As a framework author and maintainer, I want to let my users do as much as possible without having
to ask any questions, especially to ask me questions.  The less questions I receive, the better quality my work is.  
So that meant the framework had to be light, thin, obvious.  
  
The result was a complete automation framework spanning all the way from the literal test case descriptions,
to the report being delievered after executing the tests.  
At the core was a set of libraries to control the IPC communication interface, and make assertions about what happened
across that interface.  Small and highly cohesive was the design goal here, which leads to modularity so that any automation framework could use them.  
The automation I chose to use was Cucumber JVM.  So there was a small thin Cucumber layer ontop of the portable core layer.  
Small but critical.  The thinnesss comes from essentially being a wrapper driving the core layer.  But the critical part was to initialize the Cucumber framework with configuration and hook it up to dependency injection, in this case I used container.  

Then the Cucumber layer was made automatable by making it drivable by command line and configurable via config files and command line arguments.  
This allowed a cloud-based pipeline to be in control of the tests, such as Jenkins or Atlassian.  
I set up and maintained the entire pipeline ops for this system.
  
Once I had pipeline control of all execution and configuration, I was then able to do the final integration of
triggering tests and delivering reports, thus wrapping up the complete system to be delievered and used by
the Engineering department.  
  
After I proved this implementation the team leader had all of the developers running tests using this system, 
and ops people replicating the pipelines to increase coverage.  
Once I trained an initial set of people, they trained each other.  
Pretty soon people were using it and giving presentations on it and didn't know I designed and wrote the whole thing :p.

The results were great and everyone was happy.  
  
For me there was one more thing, learning about what testing really is.  
A lot of people state opinions as ultimatums and present it as advice.  
This system actually broke a lot of "rules" and I am proud to say it worked out just fine 
and don't think I would have done it any other way.  
So, trust your intuition so you can read more stuff and take in more opinions but always be solid.


