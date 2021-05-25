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
**Warning: highly opinionated post.  My perspective is from embedded systems.**

### An object file (.o) is an instance of a class?  Stay with me for a second.
In C++ a class is a data structure, ie. struct,ie. a contiguous blob of memory, plus code that operates on that memory.
In C++, defining a class does not reserve any memory.  Instantiating a class reserves the memory.
You can call the functions and access the data if they are public.  You think of the functions as being encapsulated from you
the caller, though many times it is not actually encapsulated at all (member function definitions in .hpp files).
  
In C, there are no classes, but there are object files.  Forget structs here in this paragraph, not talking about structs.  
> You can think of object files as instances of classes.
> This is not an exclusive relationship, just a loose relationship that may help draw analogies and ideas from.  


An object file also has functions and data.  You can call the functions and access the data if it is non-static.
In this mental model, an object file is 1 instance of a class.  An object file is a single object.
An object file comes from a single C source file.  So the analogy extends to, a single C source file
can be viewed as a single object, and transitively,  
  
> a single C source file can be viewed as a single instance of a class.

### Let me demonstrate this idea
  
To illustrate this, let me point out 3 different ways of doing the exact same thing.  
In C you can entirely encapsulate the definition of a struct, and a single instantiation of that struct by using "static"
ie. file scope.  Then you can have public ie non-static functions which can modify that 1 struct instance.
These functions may be called eg.  
```c
    // This is the "C file as object" style
    // When these functions are called here, no type definitions are needed
     I2C_2_Init();   
     I2C_2_Write(addr,data);  
```
    
There would be a file called i2c2.c which privately ie. static, defines the struct and instance of the struct.  
There might be another file called i2c1.c, which is the same thing but for the I2C_1 interface.  
Notice the caller only sees functions, no struct types or struct variables.  
This can be a useful idea.  I absolutely am not saying you should use this idea exclusively, it is just 1 more idea
among many other non-mutually-exclusive ideas.
  
  
### Contrast this with the other 2 styles
More commonly you see styles like below.  Below are the C and C++ equivalents.  
In C++, `i2c2` is an instance of a class.  In C, `i2c2` is a struct instance passed by reference.  
But it is entirely equivalent to the previous idea, "C files as objects" style.  
```c
    // The C++ idea
    i2c2.init();
    i2c2.write(addr,data);
    
    // The C idea
    I2C_Init(&i2c2);
    I2C_Write(&i2c2, addr, data);

```

All of the above 3 representations are the same thing at the end of the day, with very minor differences in runtime overhead.  Do you agree?  

### Why is it hard to grasp the underlying sameness of all these patterns?

In C++ the paradigm of using source and header files to create object files to then pass to the linker is entirely messed up due to the following non-exhaustive list:  
* Constructors
* Breakage of encapsulation
* Header only libraries
* Different usage of global variables
* Excessive use and reliance on templates

Although the underlying reality has not changed, it's a completely different paradigm 
to the programmer, that is so convoluted that (novice) C++ programmers
don't really have any reason to understand it, and so they don't see the fundamental C paradigm either.  

In C, it is very common to have .c files with global variables defined at the top (static ie filescope).
In C++, many .cpp files do not have global variables.
Of course the global variables have to exist somewhere but they are either class globals (ie. static in a class),
or they exist in a small set of "top level" cpp files.  
A lot of times global variables are hardly used at all because people in my opinion tend to overuse and over-rely
on dynamic memory allocation and standard algorithms.  
> Notice that the idea "C files as instances of classes" is absolutely not compatible with the ideas of dynamically instantiating classes, move semantics, value types, RAII, composition of std::algorithm's, none of that.  

Hence me saying this is just a loose analogy, not a tautology.  
  
  
This is static allocation, ie. global variables.  It's not sexy abstract cool stuff, this is the cold hard truth.  The abstractions put wool over your eyes.  Why are you even using C++ at all?  Red pill or blue pill?  I like both here.  
  
I think this is related to why I tend to meet C++ programmers that don't understand translation units.
The conceptual understanding of translation units maps well onto the C programmer's model,
but it does not for C++.  

A C programmer could think of writing all C code in assembly by hand and using the linker to link the executable.
Not that they would, but they can imagine how it would be structured and written.
A C++ programmer may get brain damage attempting the equivalent.  
  
For some reason (novice) C++ programmers think they are at an abstraction level as high up from the CPU as python is.
So high that they don't see a need to even have a programmer's model of the CPU itself.
So high that they make-believe and assume the existence of a magic layer of stuff that holds everything 
they don't understand.  But constantly the boundaries and contents of this layer morph around as
logical contradictions are forgotten and make-believed again.  Is it runtime, link time, compile time, preprocessor time, is it the OS, is it the CPU itself.  People actually think there is a C++ runtime environment like there is in java and python.  They literally lie to themselves like conspiracy theorists do.  
This is entirely wrong.  C++ is much closer to C and much closer to the CPU than python is.  
There is no magic, there is only the machine.  


### The object file as class instance idea works nicely with the filesystem
  
OK enough of the ranting.  Why do I care to share this idea?  
  
I personally like using the filesystem to be the skeleton of my architecture.  
The hierarchical view, if it maps onto the system you're developing, works well.  
So I don't necessarily see a problem with i2c1.c and i2c2.c.  
It doesn't scream out to me that there should be i2c.c, and then elsewhere 2 instances of i2c_t.  
Sure, I would not like to see massive duplication in i2c1.c and i2c2.c but it is possible to factor out dupes.  
What I do like is that I can see that there are 2 I2C interfaces being used in this system.  
Perhaps if this was all under a BSP/ layer, looking inside BSP/ would tell you all the components.  

Perhaps try thinking of an executable from the linker first, then out to the source code.  
Instead of looking at the source code and doing a hail mary hoping for an executable to come out.  

