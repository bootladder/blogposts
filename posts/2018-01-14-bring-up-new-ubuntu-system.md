---
layout: post
title: Bring up New Ubuntu System
---
Ubuntu on my laptop was crashing regularly and taking way too long to boot from the SSD.  
  
# Editing this Blog
  Need: Clone git repo, apt-get install ruby ruby-dev, gem install bundler, bundle install  
Need: Production repo URL, so it can be pushed to.  
The prod remote is a bare git repo on the production server.  
Put in the url to it?  Behind web root?  *A:  use the whole SSH URL*  
Use ~/.ssh/config to specify the key to be used.  
*New*:  The built \_site had been stored in the git repo, so it
could be directly copied into the bare remote, and then copied to the deploy.
I thought of course, to `git rm` the \_site and then run `jekyll build`
inside /deploy, but of course jekyll was not installed.  
So for now I put the built \_site back in the repo.

# Atom IDE
`sudo add-apt-repository ppa:webupd8team/atom`  
`sudo apt update; sudo apt install atom`  
`Source: http://tipsonubuntu.com/2016/08/05/install-atom-text-editor-ubuntu-16-04/`

# Aliases
```
#Steve Aliases
alias l='ls -ltrh --color=auto'
alias gs='git status'
alias gl='git log'
alias gd='git diff'
```

# Random Color Command Prompt
```
---
...inside .bashrc
---
function nextcolor
{
    mycolor=$(($RANDOM % 7 + 31))
    echo $mycolor
}
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;$(nextcolor)m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
```

# VIM
`sudo apt-get install vim`
```
~/.vimrc
set noexpandtab
set tabstop=2
set shiftwidth=2
set clipboard=unnamedplus
```

# Golang

Copy pasted from instructions: https://golang.org/doc/install?download=go1.9.2.linux-amd64.tar.gz
```
Linux, Mac OS X, and FreeBSD tarballs

Download the archive and extract it into /usr/local, creating a Go tree in /usr/local/go. For example:

tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz

(Typically these commands must be run as root or through sudo.)

Add /usr/local/go/bin to the PATH environment variable. You can do this by adding this line to your /etc/profile (for a system-wide installation) or $HOME/.profile:

export PATH=$PATH:/usr/local/go/bin
```

`go get -t -u ./...` -u installs from the net, -t installs for tests.  
**-t is not in man go get ???**
  
# Misc  
```
git clone https://github.com/sstephenson/bats.git
cd bats
./install.sh /usr/local
```

# Thunderbird
Don't download all emails from IMAP server, only last 30 days
`Source: https://support.mozilla.org/en-US/questions/1066286`  
Why doesn't this work, it downloaded all my emails.  Maybe because there weren't that many of them?

# Servers
Need SSH Keys, Hostnames, Root Passwords, IP Addresses  
_Gnome Keyring_ turns out it reads the keys stored in ~/.ssh.  Confirmed, easy, just copy the keys into there.  Continue using ~/.ssh/config to specify which key per host

# Ubuntu Crap
Disable Caps-Lock:  `sudo apt-get install gnome-tweak-tool`  , go to "Typing"  
Eable Workspaces:  `Settings-->Appearance-->Behavior-->Enable Workspaces`

# X, i3, Bind Capslock to Windows Key (Super)
setxkbmap -option caps:super


# arm-none-eabi-gcc  
`apt-get install gcc-arm-none-eabi`
  
# IRC
`apt-get install xchat-gnome`
`/msg nickserv register`

# Synergy
`https://github.com/symless/synergy-core/wiki/Compiling`  
```
 git clone https://github.com/symless/synergy-core.git
 cd synergy-core
 mkdir build
 cd build
 cmake ..
 make
```
An Ubuntu 16 laptop didn't have libcurl so,  
`apt-get install libcurl4-openssl-dev`  
Also it didn't have XKBlib.h
`apt-get install libx11-dev`  
As other people have noticed you have to edit CMakeLists.txt to change path to find XKBLib.h  
Change it to /usr/include.  
Weird, CMake still can't find it, but if i do a rm -rf build and repeat the build again it works.  
Then it can't find libxtst. `apt-get install libxtst-dev`  
That worked well, now make a config file like this:  
```
section: screens
        steve-T420:
        steve-T420-ZoZ:
        bob-pc:
end
section: links
        steve-T420:
                down = steve-T420-ZoZ
                left = bob-pc
        steve-T420-ZoZ:
                up = steve-T420
        bob-pc:
                right = steve-T420
end
```
Then run it with this  
`./synergys -f -c /etc/synergy.conf`  
Run the client like this  
` synergyc -f 10.0.1.23`

# Hopper
Just download the .deb, Ubuntu software manager fails miserably as usual, but doing this apparently makes it work  
` sudo dpkg -i Hopper-v4-4.3.12-Linux.deb `
