---
layout: post
title: Classes vs Object Files in C++ vs C
date: 2021-05-22
summary: C doesn't have classes but it does have objects.
    There is a relationship between classes and object files.
categories:
  - "C/C++"
  - "Embedded"
---

# Sorry, this post is WIP, but thanks for visiting!

In C++ a class is a data structure, ie. struct,ie. blob of memory, plus code that operates on that memory.  
In C++, defining a class does not reserve any memory.  Instantiating a class reserves the memory.  
  
In C, there are no classes, but there are object files.  You can think of object files as instances of classes.  
This is not an exclusive relationship, just a loose relationship that may help draw analogies and ideas from.  

In C you can entirely encapsulate the definition of a struct, and the instantiation of it by using "static"
ie. file scope.  
Then you can have public ie non-static functions which can modify the struct instance.  
These functions may be called eg.  I2C_2_Init(); and I2C_2_Write(addr,data);  
There would be a file called i2c2.c which privately ie. static, defines the struct and instance of the struct.  
There might be another file called i2c1.c, which is the same thing but for the I2C_1 interface.
  
In C++ you see more stuff like this:  i2c2.init() , i2c2_write(addr,data);  
Here i2c2 is an instance of a class.  
But it is entirely equivalent the C version.

In C++ the paradigm of using source and header files is entirely messed due to the following:  
Constructors, breakage of encapsulation, header only libraries, different use global variables.  
It's a completely different paradigm and you can't confuse them.  

In C, it is very common to have .c files with global variables (static ie filescope).  
In C++, many .cpp files do not have global variables.  
Of course the global variables have to exist somewhere but they are either class globals (ie. static in a class),
or they exist in a small set of "top level" cpp files.  
  
I think this is related to why I tend to meet C++ programmers that don't understand translation units.  
The conceptual understanding of translation units maps well onto the C programmer's model,
but it does not for C++.  

A C programmer could think of writing all C code in assembly by hand and using the linker to link the executable.  
Not that they would, but they can imagine how it would be structured and written.  
A C++ programmer may get brain damage attempting the equivalent.


I personally like using the filesystem to be the skeleton of my architecture.  
The hierarchical view, if it maps onto the system you're developing, works well.  
So I don't necessarily see a problem with i2c1.c and i2c2.c.  
It doesn't scream out to me that there should be i2c.c, and then elsewhere 2 instances of i2c_t.  
Sure, I would not like to see massive duplication in i2c1.c and i2c2.c but it is possible to factor out dupes.  
What I do like is that I can see that there are 2 I2C interfaces being used in this system.  
Perhaps if this was all under a BSP/ layer, looking inside BSP/ would tell you all the components.  

Perhaps try thinking of an executable from the linker first, then out to the source code.  
Instead of looking at the source code and doing a hail mary hoping for an executable to come out.  

