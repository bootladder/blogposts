---
layout: post
title:  "How to make Linux Makefiles Work Good"
author: Steven Anderson
categories:
  - "C/C++"
---
  
I'm going to describe what I do for my current project.  I'm porting over my 5th project to this system so I gotta write it down haha.  
  
Main Elements:  
VPATH:  You have to append every source directory in the VPATH.  
OBJS:  Then you list all the objects being compiled
  
Use a command like this:
OBJS_PRODUCTION_CROSS = $(addprefix build/,$(PRODUCTION_OBJ_NAMES) $(PRODUCTION_TARGET_SPECIFIC_OBJ_NAMES) $(PRODUCTION_APP_OBJ_NAMES) $(ASFNAMES) )
  
To create a list of Target Object Paths.  These are the objects you want to build.  They'll be in the /build directory.  
  
The Make targets:  For building each object use a Make target like this:
build/%.o : %.c
  @echo Building file: $<
  @echo Invoking: ARM/GNU C Compiler : 5.3.1
  $(CROSS_C_COMPILER)  -x c -mthumb $(PROJECT_VERSION_NUMBER)  $(PROJECT_FLAGS) $(PROJECT_INCLUDES) $(PROJECT_COMMON_FLAGS)  -mlong-calls -g3 -Wall -Werror -m

  
This is why you made that list of the target objects before.  That's a list of dependencies.  Every object is a dependency.  
The Make target covers all of these.  
The cool part is the dependency : %.c here.  This is where the VPATH is cool.
No need to specify where any particular source file is.  The VPATH contains all the directories with source.  
  
To build that source C file, all you need is the path to it so that's all the VPATH does.  
The rest is just obvious compiler flags!  
  
So what I"m doing is converting a Visual Studio generated makefile with all this Windows specific crap.  
Into a PORTABLE-ABLE MAKEFILE THAT I HAPPEN TO BE USING WITH LINUX.  Ahem.  
  
Let's create that PORTABLE-ABLE Makefile starter template, might as well.  
  
To get incremental progress I'll start with the all target and go from there.  
  
First the all target:  
# All Target
all: $(OUTPUT_FILE_PATH) $(ADDITIONAL_DEPENDENCIES)

$(OUTPUT_FILE_PATH): $(OBJS_PRODUCTION_CROSS) $(USER_OBJS) $(LIB_DEP) $(LINKER_SCRIPT_DEP)
  @echo Building target: $@
  @echo Invoking: ARM/GNU Linker : 5.3.1
  $(CROSS_C_COMPIL 
  
What we see here is all will make the target with name specified by variable $(OUTPUT_FILE_PATH)  
To make that target, it depends on our production objects, libraries, a linker script, and whatever else.  
Let's get that working.  
  
#
#  GENERALS
#
#
#

$(TARGET_OBJECT_BUILD_DIRECTORY)/%.o : %.c
  @echo Building file: $<
  @echo Invoking: C Compiler 
  @echo $(COMPILER_COMMAND)
  $(COMPILER_COMMAND)


# All Target
all: $(OUTPUT_FILE_PATH) $(ADDITIONAL_DEPENDENCIES)

$(OUTPUT_FILE_PATH): $(DEPENDENCIES_BEFORE_LINKING)
  @echo Building target: $@
  
  
  
Works!  
  
Now to use this, I need to create variables DEPENDENCIES_BEFORE_LINKING.
These are the objects and libraries and stuff I need, linker script etc.
If DEPENDENCIES_BEFORE_LINKING has the format described below, the provided make target works.  
  
When the list DEPENDENCIES_BEFORE_LINKING includes .o object files inside the TARGET_OBJECT_BUILD_DIRECTORY,
they will match the target.  The source file will be found from the VPATH.  
  

