---
layout: post
title: How to test getopt
categories: guide
---
  
I have a function with associated unit tests, called Parse_Bootloader_Commandline().
  int Parse_Bootloader_Commandline(int argc, char * const argv[])  
It clearly takes in the argc and argv from main(), and parses it.  
  
I had it working, and then I wanted to change the internals of that function, to use getopt.  
Interesting, I found that the unit tests were still applicable.  They had names eg.  
ParseCommandline_InvalidCommand_Returns0  
ParseCommandline_ValidLoadCommand_SetsCommandVariable  
  
That's cool!  I swapped out the internals and I know to some degree that the external users of this unit will be OK.

The new code that I swapped in, I had used before but did not have tests for it.  So I knew for example that my syntax for calling getopt was OK.  
I did have trouble compiling, because I was using -Werror and you probably know with crazy casts and use of const and pointers and arrays, you get warnings.  In my case the compiler didn't like my use of const char * argv[].  
When I got the new code to compile, the above mentions tests FAILED !  Why???  
  
I figured it was because I changed something with the types and casts to make the compiler happy.  
  
I tried a bunch of things that didn't make any difference.  Casting to (char \*\*).
  
I then found out, that, getopt uses global variables that do not get reset !!!  
Now, obviously there were global variables because you have to use them, **optarg**.  
But to me I had no idea there was another global variable,  
**optind** ,  
and this is the argument index!  Starts at the beginning of the command line and when it gets to the end, that's what makes the next getopt() call return EOF!  
  
In my tests, getopt() was being called in each test!  Of course only the first test worked!  
  
So obviously the solution is to put the following line in the TEST_START{} block or whatever you call it, it's the code that runs before every test!  
  
**optind = 0**
  
I literally added that one line and all my tests passed.  Crazy! 
