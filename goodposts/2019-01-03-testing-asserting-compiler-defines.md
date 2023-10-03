---
layout: post
title: 'Testing: Asserting Compiler Defines'
date: 2019-01-03
summary: Tests are good, but can you test for compiler defines being present?
categories:
  - Embedded
  - C/C++
---
I ported a build from make to CMake, I nearly missed a piece though just before shipping.  
The **version number** did not "get baked into the build".
The version number was stored in the build system and got put into a compiler `#define`d symbol.  
In make, variables are declared like `myvar=hello`, but in CMake it's like `set(MYVAR hello)`.  I forgot to port those over and so the compiler define flag  
   `-DVERSION_NUMBER=myversionumber`  
was not there in the build command.  
  
I had no test to tell me I forgot this, so I got lucky that I remembered to fix that before shipping.  
  
**What can I do about that:**
  
1. **Assert the defined symbol matches some magic**  
If say, your version number starts with some number, or it's a string with some prefix, then a unit test can compare the defined symbol and assert.  
I'm trying Ceedling now and 1 issue is, Ceedling builds the test exe for the host with a separate build system.  If I wrote tests asserting the compiler `#define`, I'd pass the test by passing the define in the Ceedling build, not my production build.  Still better than nothing though.
  
**This is what I came up with:**  
```
void test_VersionNumber_IsDefined(void)
{
  uint16_t version_number = VERSION_NUMBER;
  TEST_ASSERT_EQUAL_HEX16(version_number, 0xEE20);
}
```
**And in the ceedling `.project.yml`**
```
:defines:
  # in order to add common defines:
  #  1) remove the trailing [] from the :common: section
  #  2) add entries to the :common: section (e.g. :test: has TEST defined)
  :commmon: &common_defines
    - VERSION_NUMBER=0xEE20
  :test:
    - *common_defines
    - TEST
  :test_preprocess:
    - *common_defines
    - TEST

```
  
So what this does is, if I ever port my build system again, that test will fail, causing me to put in that define entry in the `.project.yml` , and then I would just tell myself to put it in the production build system also.  
  
I guess that helps a bit, and it's a super small test obviously.
