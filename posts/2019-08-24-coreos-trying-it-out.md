---
layout: post
title: 'CoreOS: Trying it out'
date: 2019-08-24 15:25 -0400
---

Installing Docker on a fresh server gets old.  Perhaps CoreOS can work out of the box?
I'm giving linode CoreOS a try.
  
1.  root user is not enabled by default, so use the core user. 
1.  indeed, Docker is installed, yay.  But no, docker-compose is not installed
  
Interesting, it says here <https://stackoverflow.com/questions/29086918/docker-compose-to-coreos>
that `docker-compose` should be run in a container, for example: 
```
$ docker run -v "$(pwd)":"$(pwd)" \
             -e DOCKER_HOST=tcp://10.0.2.15:2375 \
             -e COMPOSE_PROJECT_NAME=$(basename "$(pwd)") \
             --workdir="$(pwd)" \
             --rm \
             dduportal/docker-compose:latest up -d
```
from <https://hub.docker.com/r/dduportal/docker-compose/>

```
alias docker-compose="docker run -v \"\$(pwd)\":\"\$(pwd)\" -v /var/run/docker.sock:/var/run/docker.sock -e COMPOSE_PROJECT_NAME=\$(basename \"\$(pwd)\") -ti --rm --workdir=\"\$(pwd)\" dduportal/docker-compose:latest"' 
```

Adding an alias to `.bashrc` requires deleting the symlink in the home dir.


Let's restore one of the backed up static sites that I host.  
  
Wow another weird thing with interactive terminals.  
Before I noticed that the `sebp/lighttpd` docker image had a breaking change, now requiring `tty:true` in the `docker-compose.yml`
  
But then in the above alias, it specifies `-ti`.  Not exactly understanding but, with both `tty:true` and `ti` , `docker-compose up` does not work.  So I removed the `-i` from the alias.  And... it still doesn't work.  So I'll just ditch the `sebp/lighttpd` and switch to ... nginx?  
  
Cool, just use the default nginx image.  
  
Now I need GNU Screen.  This is interesting:  <https://unix.stackexchange.com/questions/203606/is-there-any-way-to-install-nano-on-coreos/222043>
```
toolbox
dnf -y install screen
```
But it's confusing to run screen in a container because the docker-compose alias is not available.  
  
So I ditched screen and just used `docker-compose up -d`
  
  
Now I will deploy the https-deployer.  Neat!  
  
** Lessons Learned: **
1. Always execute application code, inside containers so you can't accidentally `rm -rf` the host.  Especially when debugging!
1. Do not keep backups on the same machine as applications.
  
Thanks!
