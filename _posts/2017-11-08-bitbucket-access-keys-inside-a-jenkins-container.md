---
layout: post
title: Bitbucket Access Keys inside a Jenkins Container
---
Bit of extra work to get a container to talk to Bitbucket.  
This is necessary because Jenkins needs to pull private repos autmatically,
so must use keys.  
The key could be baked in the image but then I can't share it.  
So what I did was put in a global /etc/ssh/ssh_config that points to a key
inside the shared data volume.  I simply stick the key in there.
  

First generate the key, then copy/paste it into bitbucket repo settings access key.
Then copy the key to an accessible server.
Then copy the key into the Docker host and then docker container.


 ssh -i bitbucket_access_key git@bitbucket.org

to make sure the key works.

Then configure SSH to use the keys there.

Problem, the only shared volume is /var/jenkins_home in the container.  /etc is not persistent.
We'll have to bake it into the image?

Add this to the Dockerfile and add this file

Host bitbucket.org
    HostName bitbucket.org
    User steveODI
    IdentityFile /var/jenkins_home/bitbucket_access_key

Dockerfile:
COPY ssh_config /home/jenkins/.ssh/ssh_config

no,
COPY ssh_config /etc/ssh/ssh_config

