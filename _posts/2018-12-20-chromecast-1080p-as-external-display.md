---
layout: post
title: Chromecast 1080p as External Display
---
I got a Chromecast and it supports 1080p, so I plugged it
into a big TV and tried it out, **but**, I got way less than 1080p.  
Why?  Because my laptop is 1366x768.  
**Even worse,** it's just a mirror, not extended like an external display.
  
# The Solution has 2 parts:  1080p Display , Extended Display (not mirrored)
  
Please excuse my butchering of X terminology.
  
## 1. 1080p Display
I used `Xephyr` to create an X server in a 1900x1080 window
(the resolution of my TV).  I assigned it to display :1.  
Then I ran a command to start a window manager (awesomewm)
 and display it in that X server.  
Then I ran a command to run Chrome and display it in that X server
ie. display :1.  
 
Then I moused over in Chrome to select the Cast menu and cast to the TV.  
  
## 2. Extended Display

Just using #1 by itself was almost good because you could work
on the laptop and have something displayed on the TV like a browser tab, youtube video etc. 
But had this limitation: 
In order to focus a window being displayed on Chromecast, 
you have to display that window on the primary (ie. laptop) display,
 so both screens are displaying the same thing.  Wasteful!
  
To solve this I used `xrandr` to create a virtual display, VIRTUAL1, of 1920x1080.  
Then I used `arandr` to activate the virtual display and lay it out next to the laptop.  
  
The following parts are specific to a tiling window manager like i3 or awesome.  
I moved the Xephyr window (ie. the window with X, awesome and Chrome) 
into an empty workspace, (eg. workspace #7 using mod+shift+7).  
Then I assigned that workspace to the virtual display, VIRTUAL1. 
(this works in i3, don't know about other wm's).  
  
# The resulting flow:

* Do work on laptop
* Focus to the Chromecast with mod+7
* Gain full control of Chromecast window with CTRL+SHIFT
* Do work on Chromecast
* Release control of Chromecast window with CTRL+SHIFT
* Focus to laptop with mod+x , x is any other workspace.
* Do work on laptop


# TLDR; Commands:
```
Xephyr -br -ac -noreset -screen 1920x1080 :1
DISPLAY=:1 awesome &
DISPLAY=:1 chromium-browser &
```

`xrandr --addmode VIRTUAL1 1920x1080_60.00`
`xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync`
`cvt 1920 1080 60`
`arandr`




# BTW, the latency is horrendous.  But the throughput keeps up
It was already bad to begin with but it's even worse with this set up.  

# Dont forget the clipboard issue
Yeah, clipboard is not shared between the 2 X servers.
