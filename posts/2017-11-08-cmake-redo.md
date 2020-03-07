---
layout: post
title: CMake redo
---
Running into so many little issues.  

* External Libraries, git repos  
I have 2 libraries that are shared by multiple projects.
I've experimented with linking to a prebuilt binary that is shared by multiple projects,
or having each individual project compile the library source.  
  
Linking to prebuilt worked before when I wasn't automating my build.  
I just installed the libraries and used hard coded paths to connect everything up in my handwritten Makefile.  
Now I can't do that, I want this on Jenkins and quick deploy.  
  
So I am trying to build the source.  
I took the libraries out of the /src tree  
```
ExternalProject_Add(XYZ_COMMON_SOURCE
    GIT_REPOSITORY "https://steve@bitbucket.org/xyz/xyz_common_source.git"
    BUILD_COMMAND ""
    UPDATE_COMMAND ""
    CONFIGURE_COMMAND ""
    INSTALL_COMMAND ""
    PREFIX "/tmp/lib/"
)
```
`/tmp/lib is under the filesystem root, not a cmake root'
  
This allowed me to do this
```
file(GLOB_RECURSE SOURCES_RECURSE
    "/tmp/lib/src/XYZ_COMMON_SOURCE/*.c"
    "/tmp/lib/src/XYZ_COMMON_SOURCE/*.h"
)
```
  
# But my next problem is, some of these source files have header dependencies.  Interface header.
  
One of the files in COMMON_SOURCE has a header, "IHAL_MYSERIAL.h"
A hardware interface.  
The project I'm trying to build now doesn't have that interface or use that source file.  
But CMake wants to build all the source files, and IHAL_MYSERIAL.h isn't there.
Eh, thinking again, the header should be in there.  I put it in.  

OK got somewhere, just had to take out some files from the Primary project that
were multiply defined with the common source.  
Previously I had avoided this by individually selecting each source file to be built.  
But now I'm building everything so there are conflicts due to that.  
  
# Small Note,
If the build fails I guess the artifacts are deleted?  
I had a GNU size command running after the linker step, but it failed due to the wrong path.  
When that happened there was no binary in the bin directory.  
But removing the size command entirely, the binary was there.  Also when fixing the wrong path,
the binary was there.
