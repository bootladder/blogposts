---
layout: post
title: Configuration Management of my Laptop
---
Similar to the contact list in my phone, my main laptop is a single point of failure.  
I take my laptop on the plane, in the car, etc.  Sometimes if I'm taking my car,
I'll take a spare laptop just in case it gets stolen.  Any time I have my main laptop,
I must have it on my person at all times, even if I'm not going to use it.  
**This is just plain stupid**.  I can get a Thinkpad T420, which is what I use exclusively now,
for less than $200.  It's relatively heavy by today's standards.  Sometimes I feel tied to it
in an unpleasant way.
  
# The only reason I have a primary laptop is because it's configured to my preference.
  
I have a docking station for my T420 in my home office.  There's a lot of stuff connected to it,
so having it saves me a lot of time unplugging and plugging.  
But you know what's funny?   Docking and undocking the laptop itself is starting to take too much time.  
Realizing the laptop is not there on the docking station, and I have to go get it and dock it... is not fun.  
  
# Now that I have a supply of disposable Thinkpads, the docking station is obsolete.

So, today I went out to the coffee shop, and I took a spare Thinkpad.  Not my main one.
I left my main Thinkpad on the docking station.  I got to the coffee shop, opened up the laptop,
and then it hit me.  The spare Thinkpad wasn't configured to my liking.  
* My i3wm config was default; I could barely use it
* It didn't have any of my books on it
* golang was not installed
* none of my git repos for work projects were cloned
* No SSH keys, no ~/.ssh/config, no /etc/hosts, no nothing!
* blah blah blah, you know the deal

# What do I do about this?

# Some of the stuff is mine, not for you!  Some of the stuff is my customer's, not for me in a year from now!
  
# Solution: Run a OpenSSH server on the main laptop
Using the "fleet laptop", ssh into the main laptop to grab configuration files.
At the same time, copy them into a git repo for future convenience.
Even better, add a line in a script that copies it to the destination.  
Worked well for `/etc/` and `~/.` files.  
`.bashrc .vimrc i3config `
  
# Solution:  Ansible Playbooks
Golang is a multi-step install, and for some reason apt-get install golang-go installs version 1.6.  Too old.  
Let's give it a try.  
```
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get install ansible
```
**Maybe I should just keep a ansible binary off hand, to avoid those 3 steps**  
  
TBD...
  
# /etc/hosts and ~/.ssh/config ... Bitbucket?
Bitbucket gives you private repos.  Let's just do that for now.  
*~/.ssh/*  
Copy over my ~/.ssh into the repo, take out unnecessary stuff.  
My `~/.ssh/config` contains:  hostnames (IP addresses or domain names), IdentityFile's (SSH keys).  
Should I include the keys in the repo?  If I do, it's more convenient, but then it's more security risk.  If I don't, I need to generate new keys and then copy them to the hosts.  At best here I have to remember a password to access the remote machine.  At worst, the machine only authenticates with keys, no password login.  
So again, for lack of a better solution, I'll put my keys in the repo.  
*Result*:  `~/.ssh/` contains key-pairs and the config file.  
Then I attempted to login to a remote host.  Turns out, git does not preserve the 400 permission mode that the keys need to have.  So, in the install script, I change the keys to 400.
  
*/etc/hosts*  
Comparing my "main laptop's" `/etc/hosts` with the other laptop's default `/etc/hosts`, I notice 2nd line of the file:  
`127.0.1.1 my-hostname` .  This obviously is unique to the host.  But it appears that this doesn't matter.  I should be able to remove that line.  
OK I did that, and I notice that when /etc/hosts is touched or modified,
I get the following message: `sudo: unable to resolve host my-hostname`.  
Using this answer as a suggestion: https://askubuntu.com/a/524368 , I added the following line to my install script: `echo 127.0.1.1 $(hostname) | sudo tee -a /etc/hosts`
