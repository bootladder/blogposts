---
layout: post
title: jenkins and build inside a docker container
---
Jenkins will be available through some URL.
The build toolchain will be installed in the Dockerfile,
on top of the jenkins image.
  
Toolchain is static but the code isn't so the code building
is done inside the container.  pulling the code, dependencies,
building it, etc is a script run inside the container.
  
Installed docker on the fresh VM with the get docker script.  
Installed docker-compose also from docker.io hosted script.
  

