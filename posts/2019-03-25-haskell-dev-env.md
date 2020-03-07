---
layout: post
title: Haskell Dev Env Notes
date: 2019-03-24 23:45 -0400
---
### Building stuff in a container and bringing it out to the host, blah  

I'm a haskell noob but installing packages is confusing and doesn't work a lot for me.  And if I switch machines it's unlikely that I'll be able to reproduce the same environment.  So, use Docker!  
  
```
from haskell:latest
RUN cabal update
RUN cabal install happy
RUN cabal install stylish-haskell
```
Now I have a stylish-haskell binary in the container.  Let's bring it out:  
```
sudo docker run --rm -it -v /tmp/:/opt/src haskell-dev-env /bin/bash
```
and then `cp root/.cabal/bin/stylish-haskell /opt/src` ,  
then exit the container and the binary is in /tmp.  
For some reason this works, whereas just trying to do `cabal install stylish-haskell` on my host failed to build.  
  
Aha, see here, I like this:  
```
steve@McBain:~ $$$ ldd /usr/bin/stylish-haskell 
	linux-vdso.so.1 =>  (0x00007ffcb73f0000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f15f9f86000)
	librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f15f9d7e000)
	libutil.so.1 => /lib/x86_64-linux-gnu/libutil.so.1 (0x00007f15f9b7b000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f15f9977000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f15f975a000)
	libgmp.so.10 => /usr/lib/x86_64-linux-gnu/libgmp.so.10 (0x00007f15f94da000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f15f9110000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f15fa28f000)
```
Somehow there's a messy web of dependencies when compiling but the resulting binary is clean, linkabilitywise.  Don't get it, but works for me!

&nbsp;
### How to run Haskell in the container
Cool I figured something, it's a bash function:  
```
runhaskelldocker() {
  sudo docker run --rm -v $(pwd):/opt/app -w /opt/app haskell-1 runhaskell $@
}

```
&nbsp;
If your current directory looks like this:  
```
steve@McBain:~/prog/haskell/nginx-log-analysis $$$ ls
access.log  input.txt  main.hs  practice.hs  scrap.hs
```
Then you can run this  
```
runhaskelldocker practice.hs input.txt
```
&nbsp;  
And you can use your Dockerfile to do your stack and cabal
