---
layout: post
title: Docker Swarm Beaglebone
---
# First the Swarm Manager
Physical access to the Beaglebone allowed a startup script to be installed,
which would set up a reverse SSH tunnel to an Internet host.  
Then, install and configure the Docker Swarm.  
  
The Swarm Manager needs to have a static IP on the LAN,
so the Swarm Nodes can hard code that static IP.  
  
# The Swarm Nodes will all use the same SD card image
The filesystem in that SD card image can have a startup script with that
Manager Static IP hard coded in.  

