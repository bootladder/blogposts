---
layout: post
title: HTTPS behind a Reverse Proxy
date: 2019-04-10 23:07 -0400
---
# bjalk;jsak
&nbsp;  
&nbsp;  
&nbsp;  
After setting up logs from my nginx-proxy, I want to put all of my services
behind the proxy now, for easy central logging.  
The static sites I host that are on the same host as nginx-proxy are easy
obviously, as they are containers with `VIRTUAL_HOST` already set up.  


&nbsp;  
&nbsp;  

But I have this other web app, <freejekyllbuilder.com> which is not on the same host.  
So I need to configure `nginx.conf**  to pass that through.  
  
**But I want HTTPS on all my sites**  
`nginx-proxy** has a convenient autogenerator for SSL certs, which I used for the
static sites on that host.  
**But how to do HTTPS when proxying to a different host?**   
Could do HTTPS on the front end ie. at the reverse proxy,
and then proxy to HTTP.  That's easy but then I'm leaving port 80 open on that
other host.  
**2 options: **
# 2 levels of HTTPS, 1 from user to proxy, other from proxy to host
# Firewall allowing only the reverse proxy host to access
&nbsp;  
# Which then brings in the obvious:  put the web app on lambda?
The web app doesn't really have state so should work

# Nevermind, it's easier to put separate nginx-proxy's on each host
Then it's easy to copy the snippet for `nginx.conf` to send syslogs to the log.  
# Now it becomes DNS level for configuration


&nbsp;  
