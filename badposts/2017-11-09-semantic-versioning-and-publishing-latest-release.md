---
layout: post
title: Name Versioning and Publishing Latest Release
---
After I got Jenkins building my code, there was no semantic versioning so this is what happened:  
The built executable would have the same name every build so got overwritten.  
I had a version number but more at a higher level, like a release number.  
Now I need something to identify each build, ie. each commit.  
  
So I did this in CMake:  
```
...
set(THIS_BUILD_VERSION 0x2B30)
...
add_executable(myexename ${SOURCES} ${SOURCES_RECURSE} )

EXEC_PROGRAM(
"echo $(git rev-parse --abbrev-ref HEAD)_$(git rev-parse --short HEAD)" 
OUTPUT_VARIABLE gitrepoinfo
)

set(output_exe_filename MY_EXE_PREFIX_${THIS_BUILD_VERSION}_${gitrepoinfo})
set_target_properties(myexename PROPERTIES OUTPUT_NAME ${output_exe_filename})
```

EXEC_PROGRAM is what worked for me , not the other execute_process or whatever.  
I echo 2 git commands to get the branch name and short commit ID.  

Then I set the variable for the executable's filename, and tell the target to be named by it.  

  
This worked, so now every time I git push I get a new build identifying that commit.  
Now the problem is, the filenames are too long to type by hand so I need something 
like "latest-version.out" as an easy reference.  
I tried to use symlinks, that didn't work.  
  
Simple solution for now is to have Jenkins zip up the build, name it latest.zip ,
and publish that along with the non-zipped originals.  
That way the zip is always overwritten with the latest.zip, and the originals are still
piling up for backup.
