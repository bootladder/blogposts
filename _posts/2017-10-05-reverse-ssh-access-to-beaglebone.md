---
layout: post
title: Reverse SSH access to beaglebone
---
I'm leaving for the weekend and I want access to my 2 beaglebones.  
1 is my system under test, the other controls a USB programmer
that can flash a microcontroller on the SUT.
  
Following this guide http://xmodulo.com/access-linux-server-behind-nat-reverse-ssh-tunnel.html
  
First I'll locally SSH into the beaglebone.  

```
homeserver~$ ssh -fN -R 10022:localhost:22 relayserver_user@1.1.1.1 
```
  
* -R for reverse tunnel.  Port 10022 on remote host is forwarded to port 22 on beaglebone
* 10022 is arbitrary
* port 22 is the SSH port that beaglebone sshd is listening on
* -f for background
* -N for "don't execute a command"
  
Note:  I forgot that my VPS had disabled password logins; only permitted by key.  
I had only installed 1 key on the VPS:  for my main laptop.  
So to install 2 more keys, I SSHed into the VPS with my main laptop.  
Temporarily allowed password logins.  
Then used ssh-copy-id to install the keys on VPS.  
Finally, disable password logins again.  
  
Last step is to add the command in rc.local.
  
# Couple extra details
  
* One of the beaglebones was running SSH on port 6000.  So, I had to change the 22 to a 6000.
```
homeserver~$ ssh -fN -R 10022:localhost:6000 relayserver_user@1.1.1.1 
```
  
  
* One of the beaglebones was only accepting SSH logins by key, but it only accepted one of a static set of keys.
  
This was annoying, I had to scp the key over to the VPS.  Then to login to the beaglebone thru the tunnel,
I had to supply the key with ssh -i.
  
# Important detail for having 2+ tunnels
  
The port 10022 is arbitrary but if there are 2 tunnels they can't be both on 10022.  
So actually one of them was on 10023.  
When SSHing through the tunnel I specified which tunnel by specifying the port.
  
# Dang, the SSH connection disconnected overnight.
  
Using AutoSSH 
https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-autossh/
  
Looks promising, I just used standard config.  
Let's see if it stays up!
  


