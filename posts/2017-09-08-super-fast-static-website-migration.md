---
layout: page
title: super fast static website migration
---
A friend was paying a company a crazy amount of money per month to create a static site,
and host it.  
Friend realized they were being ripped off and stopped paying.
The company then shutdown the website host, blocking access to the site source.
  
Fortunately there was a copy on the wayback machine.  
  
I ripped the last good copy and dumped it onto my own host.
  
It was super easy and fast, took just a few hours to figure out a couple things.
  
The entire website was roughly 1MB.

* Actually you don't even need a Dockerfile, you can work straight from this image  
```
    from sebp/lighttpd
```

* docker-compose.yaml  
```
    version: '2'
    services:
      lighttpd:
        build: .
        environment:
          - VIRTUAL_HOST=mywebsite.com
        ports:
          - "14001:80"
        volumes:
          - ./website/:/var/www/localhost/htdocs

    networks:
      default:
        external:
          name: nginx-proxy
```
     
# Explanation:
* the environment variable VIRTUAL_HOST is for the nginx reverse proxy.  
  It's name based so when you HTTP request the reverse proxy looking for hostname
  mywebsite.com , the reverse proxy directs traffic to this container we're discussing.
* ports: the container lighttpd serves on port 80.  14001 is a random number.
* volumes: ./website , ie. the current directory where the docker-compose.yaml file is,
  is mapped to the web root of the lighttpd server.
  Just stick the website in there.
  
# To grab the website from Wayback Machine, I used Wayback Machine Downloader
<https://github.com/hartator/wayback-machine-downloader>
  
* Was super easy to use, just took a minute to figure out the --to and --from flags.
* When the site you're interested in is currently pointing to a domain parking spot,
  you want to get files up to a certain date.  
* If you want to exclude older files you'll have to do that explicitly also.
