---
layout: post
title: Janus Notes
---
It appears that Janus is what I want.  I may have to apply browser-side recording, but first I'll see how far Janus goes with server side.  
Apparently, browsers can connect peer to peer to a Janus server.  
  
Obviously with all the acquisitions happening in this space, I'd like my app to be independent of Janus, source-wise and functionality-wise.  

Well I guess the first thing to do is install it on a server.  
Let's compile it for fun.  
  
# Compiling Janus (from the README)
```
git clone https://github.com/meetecho/janus-gateway.git
cd janus-gateway
sh autogen.sh
```
My debian 8 VPS didn't have autoreconf.  `sudo apt-get install autoconf`  
Then got this error `configure.ac:5: error: possibly undefined macro: AC_ENABLE_SHARED`  , `sudo apt-get install libtool`  `sudo apt-get install gettext` `sudo apt-get install pkg-config`  
`./configure --prefix=/opt/janus --disable-websockets --disable-data-channels --disable-rabbitmq --disable-mqtt`  
Now tells me the packages I'm missing.
```
No package 'glib-2.0' found
No package 'nice' found
No package 'jansson' found
No package 'libssl' found
No package 'libcrypto' found
```
Hmm, didn't know about jansson, that's cool, a JSON library in C.
```
sudo apt install glib-2.0
sudo apt install libjansson-dev
sudo apt install libnice-dev
```
Oh, somehow I missed this, I'll just run the one liner from the README
```
	aptitude install libmicrohttpd-dev libjansson-dev libnice-dev \
		libssl-dev libsrtp-dev libsofia-sip-ua-dev libglib2.0-dev \
		libopus-dev libogg-dev libcurl4-openssl-dev pkg-config gengetopt \
		libtool automake
```
As mentioned in README, apt install libsrtp didn't work.
```
	wget https://github.com/cisco/libsrtp/archive/v1.5.4.tar.gz
	tar xfv v1.5.4.tar.gz
	cd libsrtp-1.5.4
	./configure --prefix=/usr --enable-openssl
	make shared_library && sudo make install
```
Nice, here's my configure output
```
libsrtp version:           1.5.x
SSL/crypto library:        OpenSSL
DTLS set-timeout:          not available
DataChannels support:      no
Recordings post-processor: no
TURN REST API client:      yes
Doxygen documentation:     no
Transports:
    REST (HTTP/HTTPS):     yes
    WebSockets:            no
    RabbitMQ:              no
    MQTT:                  no
    Unix Sockets:          yes
Plugins:
    Echo Test:             yes
    Streaming:             yes
    Video Call:            yes
    SIP Gateway (Sofia):   yes
    SIP Gateway (libre):   no
    NoSIP (RTP Bridge):    yes
    Audio Bridge:          yes
    Video Room:            yes
    Voice Mail:            yes
    Record&Play:           yes
    Text Room:             yes
Event handlers:
    Sample event handler:  yes
    RabbitMQ event handler:no
JavaScript modules:        no
```
Alright, a make, sudo make install, sudo make configs all worked!
  
OK, now how do I connect to Janus?  
  
# Connecting to Janus
  
I'm running the audiobridge demo.  The audiobridge.js has a hard coded URL for the Janus server.  I'm using this locally, ie. file:// , which I know only works on Firefox right now.  But this code won't work...  
```
var server = null;
if(window.location.protocol === 'http:')
  server = "http://" + window.location.hostname + ":8088/janus";
else
  server = "https://" + window.location.hostname + ":8089/janus";
```
So I will change this to set server = http://myexplicitURL:8088 ?  
  
Oh, first I have to run Janus.  It says `HTTP webserver started (port 8088, /janus path listener)...`
Oh nice, some debugging help.  I point my browser to mydomain:8088 and it says Invalid URL.  Then I try mydomain:8088/janus and I get Invalid session.  That looks good!  Let's change the audiobridge.js to that.
  
Oh snap!  Getting some good logs in the console.  
Dang, I can't connect another peer because I loaded this locally.  Instead of bothering to copy the files to another machine, I should just serve them.
  
# Connecting to Janus with 2 peers
So, not surpisingly, I'm getting issues with HTTPS.  Somehow I have to get it right, Janus over HTTPS, webserver over HTTPS, HTTPS proxy to a Janus over HTTP, whatever.  Well, I figured out how to get Beego framework to serve static pages over HTTPS, so that'll do.  Don't know what to do with Janus.  Let's just put the static page in a running Beego server and see what happens.
