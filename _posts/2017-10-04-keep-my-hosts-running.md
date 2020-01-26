---
layout: post
title: Keep my hosts running
---
Now that I temporarily host my friend's site who actually wants to have visitors,
I can't let it go down.  Also this blog, might as well keep it up.
  
So first, when the VPS host goes down and reboots my VPS, my VPS has to start up.

  
* Reverse Proxy, blog, friend's site
  
systemd service looks OK but... feel safer with a shell script.
  
To get this going, I'll reboot the VPS and repeat the steps.  
  
~~~
service docker start;
cd /opt/nginx-proxy/ ; docker-compose up &
cd /opt/deploy/ ; docker-compose up &
~~~
  
stick it in rc.local
  
nice!  
  
Now how about a way to check if the website ever goes down?  
Well if the host goes down it can't tell me about it.  
So, it could be the stopping of keepalive messages out of the host,
or a different host being unable to connect to the host.
  
eh, leave that one at that for now.



* l
  

