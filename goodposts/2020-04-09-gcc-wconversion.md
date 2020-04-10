---
layout: post
title: GCC -Wconversion: Warnings Everywhere!
---
# -Wall and -Wextra are actually not, all and extra.
  
It started in this test
```
:::c++
TEST(LFO, handlesMIDIMessages) {
    LFO lfo("name");

    int f = lfo.getLFOFrequency();

    // changes the frequency to some value
    lfo.setMIDIParameter(PARAM_0, 30);

    ASSERT_NE(f, lfo.getLFOFrequency());
}
```

See `int f` there.  I had just changed `getLFOFrequency()` to return a float.  But why did I not get a warning for assigning float to int?  
Well I don't know WHY, but atleast everyone else is in the same boat:  <https://stackoverflow.com/questions/24915712/assigning-float-into-int-variable-causes-no-warning>.  
  
So the answer is **you should use `-Wconversion`**.  What comes next is SHOCKING!  
  
(not really, I just added suspense)  
  
<br><br>
# These are the warnings that came up when I enabled -Wconversion

```
:::c++
  // USING THE DAFX Formula
    float thresh = clipping_percent * 32768.0;     //WARNING
    for(uint32_t i=0; i<num_samples; i++){
        if(my_abs(inputBuffer[i]) < 0.33*thresh){
          outputBuffer[i] = inputBuffer[i] * 2;
        }
        else if(my_abs(inputBuffer[i]) < 0.66){
          float inputScaled = inputBuffer[i] / 32768.0;  //WARNING
          float outputScaled =
            (3.0 - ((2 - (3.0*inputScaled)) * (2 - (3.0*inputScaled)))) / 3.0;   //WARNING

          outputBuffer[i] = outputScaled * 32768.0;
        }
        else {
          if(inputBuffer[i] < 0)
            outputBuffer[i] = -1 * thresh;
          else if(inputBuffer[i] > 0){
            outputBuffer[i] = thresh;
          }
          
        }
    }
```

Here there are 3 assignments to floats and they are all : `warning: conversion to ‘float’ from ‘double’ may alter its value [-Wfloat-conversion]`
    
Floating point literals are `doubles`, and to make them floats you can say for example `0.1f`.  So should I add `f` to all of my literals?  At first I said yes, I need to use floats, because I have a 32-bit processor with a FPU and I want 32-bit floating point numbers, ie `float`s.  
But then I realized something:  I'm using a Cortex M7 with a double precision floating point unit.  That means I can use doubles!  
Continuing on, it turns out, that my sample buffers which were `float`s, can now be doubles also.  In fact, I **should** use either exclusively doubles or exclusively floats because converting between them requires a conversion instruction.  
  

I had originally made up a `typedef float sample_t` because I wasn't sure if I wanted to do floating or fixed point.  I went with floating point and now
I'm really glad I did the typedef because now I can just change it to `typedef double sample_t` and that changes in hundreds of places.
<br>
# See this excellent post to learn more
<https://mcuoneclipse.com/2019/03/29/be-aware-floating-point-operations-on-arm-cortex-m4f/>
  
<br><br>
# Another Example
So changing `float` to `double` removes the `-Wconversion` warnings, but what if I actually did want to run this code using floats, perhaps on a Cortex-M4 with a single precision FPU?  The below uses a bunch of constants and all of them would have to be changed to float literals.  Not only to avoid the `-Wconversion` warning but actually to avoid overhead converting between `double` and `float`.  Does that mean I have to make all the literals float literals __everywhere__?  There are lots of them, plus it's annoying to see the `f`.  
You know what, while I'm at it, let me add another requirement.  I want all those constants to be evalutated at compile time.  

```
:::c++
  void setMIDIParameter(int id, int value){
    float delayNumSamples_float = ((float)value/128.0) * 100.0 * (1.0/1000.0) * (48000.0);
```  
  
To summarize:  No warnings when used with either float or double, No float to double conversion (see the blog post), all the constants evaluated at compile time.

```
:::c++
typedef float sample_t;
int delayNumSamples(int value){
    #define MAX_DELAY_MILLIS 100.0
    #define MILLIS_TO_SECONDS (1.0/1000.0)
    #define SAMPLES_PER_SECOND 48000.0
    #define MAX_INPUT_VALUE 128.0
    const sample_t VALUE_TO_SAMPLES = (MAX_DELAY_MILLIS * MILLIS_TO_SECONDS * SAMPLES_PER_SECOND / MAX_INPUT_VALUE);

    sample_t delayNumSamples_float = (sample_t)value * VALUE_TO_SAMPLES;
    return (int)delayNumSamples_float;
}
```

That is what I came up with, resulting in 
```
delayNumSamples(int):
        vmov    s15, r0 @ int
        vldr.64 d6, .L3
        vcvt.f64.s32    d7, s15
        vmul.f64        d7, d7, d6
        vcvt.s32.f64    s15, d7
        vmov    r0, s15 @ int
        bx      lr
.L3:
        .word   0
        .word   1078116352
```
For double, and,  
``` 
delayNumSamples(int):
        vmov    s15, r0 @ int
        vldr.32 s14, .L3
        vcvt.f32.s32    s15, s15
        vmul.f32        s15, s15, s14
        vcvt.s32.f32    s15, s15
        vmov    r0, s15 @ int
        bx      lr
.L3:
        .word   1108738048
```
for float.  
  
Neat, in both cases all it does is load the constant, convert the `int` argument, multiply, convert the return value.  
Interestingly I don't have to make the literals `float`'s.  If I assign to a const float, ie. `const sample_t VALUE_TO_SAMPLES`, it doesn't give the warning.
  
And, no compiler warning.  
All of this goes to show that...  
# Compiler warnings are super important
I avoided a potential bug, made my code more portable, and made it a lot more efficient, learned something about const, looked at some assembly, all as a result of satisfying `-Wconversion`.  
  
  
  
Here's another one
```
:::c++
typedef double sample_t;
#include <stdint.h>
//globals for demo purposes
uint32_t num_samples = 1024;
int delayNumSamples = 10;
int delayNumSamples_lastTimeProcessed = 2;
int pointless = 0;
int i = 3;

void process(void){
        float slope = (float)(delayNumSamples - delayNumSamples_lastTimeProcessed) / (float)num_samples;
        int interpolatedDelayNumSamples = delayNumSamples_lastTimeProcessed + (int)((float)i*slope);
        pointless = interpolatedDelayNumSamples;  //to prevent optimization
}
```
Remember from school, the slope of a line is dy/dx.  We only have integers.  
That is an ugly number of casts.  
i is cast to float because of : `warning: conversion from 'uint32_t' {aka 'long unsigned int'} to 'float' may change value`  
`i*slope` is cast back to int because of: `warning: conversion from 'float' to 'int' may change value [-Wfloat-conversion]`  
  
It satisfies the warnings but look at the assembly:  
```
process():
        ldr     r1, .L3
        ldr     r0, .L3+4
        vldr.32 s12, [r1, #8]     @ int
        vldr.32 s13, [r1, #12]    @ int
        ldrd    r3, r2, [r1]
        subs    r2, r2, r3
        vmov    s15, r2 @ int
        vcvt.f32.u32    s12, s12
        vcvt.f32.s32    s14, s15
        vcvt.f32.s32    s13, s13
        vdiv.f32        s15, s14, s12
        vmul.f32        s15, s15, s13
        vcvt.s32.f32    s15, s15
        vmov    r2, s15 @ int
        add     r2, r2, r3
        str     r2, [r0]
        bx      lr
.L3:
        .word   .LANCHOR0
        .word   .LANCHOR1
delayNumSamples_lastTimeProcessed:
        .word   2
delayNumSamples:
        .word   10
num_samples:
        .word   1024
i:
        .word   3
pointless:
```
  
2 lines of C code using 4 globals takes that much assembly.  Could have been worse.  Can it be any better?  You tell me.  We see the cost of using floats and ints interchangably.  
3 int to float conversions and 1 float to int conversion.  As this is an array operation, it adds significant overhead.  
So, it looks ugly and the ugliness itself reveals a performance hit.  


<br><br>
# Another one.  Short ints can easily be overflowed
```
:::c++
      int sawToothValue = (-127) + (127*ticks*2/T);
      currentLFOValue = abs(sawToothValue);
      midiMessage.value = currentLFOValue;  //warning: conversion to ‘uint8_t {aka unsigned char}’ from ‘int’ may alter its value [-Wconversion]
```
The thing to note here is that the value inside of a MIDI message is a 8 bit value.  But `currentLFOValue` is an `int`.  It's extremely easy to overflow a `uint8_t` when you're doing calculations with `int`.  
My first reaction is I like putting the explicit cast from int to uint8_t, rather than changing the type of `currentLFOValue` to uint8_t.  It's more explicit,
at the place of assignment, where it could go wrong.
  
```
:::c++
      int sawToothValue = (-127) + (127*ticks*2/T);
      currentLFOValue = abs(sawToothValue);
      midiMessage.value = (uint8_t) currentLFOValue;
```
   
  
<br><br>
# Not even the seemingly trivial are excepted.  
```
:::c++
    float lfoFreqHz;

...
    void setMIDIParameter(int id, int value){
        lfoFreqHz = value/10.0; //warning: conversion to ‘float’ from ‘double’ may alter its value [-Wfloat-conversion]
    }
```
The literal `10.0` is a double.  Same problem as before.  Do I make it `10.0f`, make `lfoFreqHz` a `double`, or what?  
Again I like changing `float` to `sample_t`, and making the literal `10.0` a `const`.  It should have a name anyway, I was just being lazy and heuristically picked a number.
  
```
:::c++
  typedef double sample_t;

  sample_t lfoFreqHz;

  void setMIDIParameter(int id, int value){
        const sample_t MIDI_VALUES_PER_HZ = 10.0;
        lfoFreqHz = (sample_t)value/MIDI_VALUES_PER_HZ;
  }
```
<br><br>
# Breaking News:  4 Asserts broken in Unit Test Suite resulting from -Wconversion fixes
Very odd errors too, off by 1 errors.
I would first think I accidentally deleted a line or changed a number.

Narrowing it down, I found it was due to the change from float to `typedef double sample_t;`  
**Why would changing from float to double give off by 1 errors?**  
  
<img src="/assets/compilerexplorerdoublefloatissue.png" style="width:100%" />
<img src="/assets/billtedwoah.jpg" style="width:100%" />  
  
<br><br>
# Conclusion:  Use `-Wconversion`  
  
Thanks!
