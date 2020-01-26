---
layout: post
title: Ubuntu WiFi Bridge to the Lab LAN
---
I don't have Ethernet access to my gateway router; only wifi.  In my lab area I have a few Beaglebones on a switch.  I need to be able to access them from my laptops.  Ubuntu has a convenient wifi to ethernet bridge.
**Recap: to enable the bridge** 
network manager applet "edit connections", add, IPv4 settings method: shared to other computers".  
  
**I wish I knew how to set this up manually**
* Only 1 choice of subnet for the Ethernet side:  10.42.0.0 , it's hard coded in networkmanager
  
  
OK that's the recap.  The interesting part in this post is, I just replaced the bridge laptop with a different one.  Setting up the bridge like above is simple, but now my problem is, my dev laptop can't access the newly set up laptop.  
Previously what I did was, configure `~/.ssh/config` to name the bridge laptop, and also named all the hosts behind the bridge (the Beaglebones).  To make this work I used the ProxyCommand feature of ssh config.  
  
So 1 obvious option is to rewrite all the entries in ssh config.  
But another cool option would be to set up some routing.  I suppose, the bridge laptop could push routes to everybody, saying that the 10.42.0.0 network is accessible through the 10.0.1.X, the IP address of the bridge on the WiFi subnet.  
**Woops, push routes is a OpenVPN thing.  lol!**  
  
To experiment, I added a static route on my dev laptop: `ip route add 10.42.0.0/24 via 10.0.1.25 dev eth0` , and I can ping the laptop on 10.42.0.1.  Cool!  But, if I ping a beaglebone at 10.42.0.61, I get `Destination Port Unreachable`.  Using tcpdump and checking the iptables on the bridge laptop, it appears that the bridge is not forwarding the ping to the Beaglebone.  
Hmm, I don't want to have to configure IP tables.  Let's look at the other option, rewriting `~/.ssh/config`  
  
Oh, this is it.  Simple.  
```
#######
# Lab #
#
Host zoz-woodland
  Hostname 10.0.1.24
  User steve

Host buster-woodland lab-gateway
    Hostname 10.0.1.25
    User steve

Host ffnbeagle-woodland
  User debian
  ProxyCommand ssh lab-gateway nc 10.42.0.61 22
```
To swap the gateway laptop, just move the lab-gateway alias from one host to another.  Recall, gateway laptop is the wifi-to-ethernet bridge to the lab LAN.
