---
layout: post
title: Hosting on docker behind a reverse proxy
categories: guide
---

Needed to redo my personal infrastructure.  My infrastructure at work absolutely needs work
but that's out of my control.  But my personal stuff can complement my work stuff.

Currently thinking about integrating:  blog, wiki, VPN, storage, CI server, etc.
And I would like some resources, particularly storage to be hostable on physical hardware in a home/office LAN.

And in some way that isn't a pile of mess and I can actually see what I have and what's happening.
  

First thing I did was point all my DNS at a reverse proxy server.  
I wanted the RPS to run in a docker container.  
First attempted to install docker on my $12/year VPS.  Turns out it was a OpenVZ VPS which did not have the minimum kernel 3.10 for docker.  
So had to get a $15/year KVM VPS.  Cheap but only 5G disk.  Makes sense to be primarly a proxy and possibly the VPN server; not sure about VPN speed on a VPS...

Used jwilder/nginx-proxy on docker hub.
Works great, also can add your own config for a host.  For example I want some hosts behind the proxy to be external to the proxy machine, ie. not running in a container inside the proxy machine.  
  
Then I wanted to host this blog behind the proxy.
lighttpd install.  This particular one has a VOLUME of its data directory.
So I just use docker-compose.yaml to map the volume to where the static site is stored.

I tried putting my wiki behind the proxy but was getting weird behavior most probably related to URLs or HTTP headers.  
Actually it mostly worked but logging in made it act weird.
  
Now that I could host the site in a docker container and have it behind the proxy,
I had to figure how to deploy changes.  
Found a solution, though it has coupling not sure how to avoid... chef/puppet?

I create a git repository containing:
the static site source code, and the built site.
a docker-compose.yaml and Dockerfile

  
to develop I have a laptop that can build jekyll, so I pull the repo, edit/new files, 
jekyll build, push the repo which includes the built site.

Then on my production host I run git pull to pull the built site.
Then docker-compose up serves the site.  
  
I don't even have to touch docker!  I just go to production host where the repo is, git pull, and since the volume is mapped, the changes show immediately.  

There is coupling to the reverse proxy; a docker network.  
  

