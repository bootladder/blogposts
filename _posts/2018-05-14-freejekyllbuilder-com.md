---
layout: post
title: freejekyllbuilder.com
---
# It should be obvious what this site is.
  
**I did get a quick proof of concept going.**
* Flask for the HTTP server has /upload and /download endpoints
* upload is a POST with a file.  Flask saves the file and calls a shell script on it
* upload URL is /upload/usergeneratedname123
* That shell script does a jekyll build on the file
* The file must be a zip of the jekyll root
* The shell script zips up the generated `_site` and saves it
* The /download endpoint is /download/usergeneratedname123
* User chooses usergeneratedname123 for now.

**How to Host freejekyllbuilder.com**
* I use my docker-compose setup for my interpretation of Clean Architecture
* The Dockerfile has a Flask install.  I also added a Jekyll install to it.
* A set of gems also are installed with the Dockerfile.
* The current idea is to have a fixed set of gems that people are allowed to use.
    People can use Gemfiles
  
**How to Upload**  
This is tricky with the command line.  For the prototype, I used this:  
`curl -F 'lolzzzzz=@/tmp/source.zip' freejekyllbuilder.com:9004/upload/site1`  
I really dislike the syntax for curl's -F flag.
  
**How to Download**  
This is easy, `wget freejekyllbuilder.com:9004/download/site1`  

**How would this work in a CI/CD pipeline?**  
* Someone pushes to the repo with blog posts
* A (webhook to a different server, or Travis) causes the jekyll source to be zipped up and uploaded to freejekyllbuilder.com
* freejekyllbuilder.com takes a few seconds to build
* Another (webhook or Travis) hits the production web server, which tells the server to update itself.
* The server may know the URL or can be given the URL to freejekyllbuilder.com
* The server downloads the built site, unzips, deploys into the web server root.
  
**Port 80 Issue**  
I have 50GB disk on this VPS, sweet!  I don't want to get another VPS just for freejekyllbuilder.  
I also don't want to tie up port 80 for freejekyllbuilder.  
So I will reverse proxy the port 80 on my VPS.
  
**SSL**  
I have to generate certs
  
**Meta:  How to build freejekyllbuilder.com with freejekyllbuilder.com**  
* freejekyllbuilder.com is a Docker image running a Flask server.  
* Flask happens to serve a index.html static site.
* The static site must be built from source and then deployed into this Flask web root
* So, freejekyllbuilder.com itself is a git repo which has the Dockerfile
* freejekyllbuilder-website is the source for the website.
**The flow:**
* freejekyllbuilder-website is updated
* webhook to Travis
* Travis zips the site, curls it to freejekyllbuilder.com
* Travis waits for completion
* Travis hits a webhook on freejekyllbuilder.com:9009 , some random port where the deployer listens.
* freejekyllbuilder.com:9009 knows to get the built site from freejekyllbuilder.com/download/metaself and deploy

