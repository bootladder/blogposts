---
layout: post
title: Use Phone WiFi Hotspot as default gateway for LAN
---
Big telecom companies are garbage.  Turd sandwich or giant douche?  
Comcast: 1 year contract for $100/mo, 50Mb down.  No thanks!  
Let's look into mobile internet with unlimited data.  Let's see if I can do this with my phone.  
  
* First I grab an extra laptop that has both Ethernet and WiFi.  This is the gateway
* I turn on my phone's hotspot and connect to it with the gateway laptop
* Verify I can ping the gateway laptop from my tester client
* Change default gateway on tester client to 10.0.0.6, verify Internet is now unavailable
* On the gateway laptop, run this
  
~~~
sudo sysctl -w net.ipv4.ip_forward=1
~~~
  
Now Internet is available through gateway laptop, but the gateway now has to go out the WiFi hotspot.
  
Turns out there are 2 0.0.0.0 entries in the route table.  Let's delete the one for Ethernet and keep the WiFi one.
  
~~~
route del default gateway x.x.x.x
~~~
  
  
Hmm, ping doesn't work.  Just hangs, no ouptut.  I see what's happening using tcpdump ... traffic starts at the tester client, gets to the gateway on the Ethernet interface, then goes out the WiFi interface, but the source address is still the tester client's source.  
If I run a ping on the gateway laptop, it does go through.  The obvious difference is the source address.  Let's change that.  
 
Let's try this one
~~~
# iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth1 -j SNAT --to-source 1.2.3.4
~~~ 
  
# Wow, that made it work!
