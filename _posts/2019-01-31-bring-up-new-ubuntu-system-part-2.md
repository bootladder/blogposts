---
layout: post
title: Bring up New Ubuntu System Part 2
---
# Got a new laptop, here's the step by step.
sudo apt-get upgrade
sudo apt install i3
logout, login with i3

switch to original system and backup /etc/hosts , ~/.ssh/config

sudo apt install git
git clone https://me@bitbucket.org/me/mysecrets
cd mysecrets
bash install.sh
sudo rsync -nazhvR e "sudo -u steve ssh" backups:/home/backups/etc . # need this on github
sudo passwd
#enter the password
su
#enter the password
mv /etc /orig_etc  
mv home/backups/etc /etc
mv /orig_etc/fstab /etc/fstab















# Cool Idea:  Startup Script
If I use a startup script to start applications,
then when I get a new laptop, just run the startup script,
and any software that isn't installed or configured will 
cause a failure in the startup script
  
  
# Crap.  I moved /etc and I broke the sudoers file
The only option will be to boot the live USB, mount the filesystem and fix it.  Next time make sure you've set a root password, next time move /etc over while being root.

# More crap.  /etc/fstab had block device IDs in it, meaning it cant be ported
And... /etc/hostname to.  
Ah crap, the ownership wasn't preserved for root owned files.  I can just chown -R root:root, but there's groups lp, shadow
