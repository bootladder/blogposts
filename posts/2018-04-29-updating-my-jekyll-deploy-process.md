---
layout: post
title: Updating my Jekyll deploy process
---

Before what I did was, build the site, then commit the site to git, then push the built site to the production server.  Problem is committing the built site, it's a waste.  It's better to not store the generated site.  
  
**Option:  Push to Prod, Build on Prod, Push to Github for archiving**
* Requires Jekyll install on prod.  Prod may not have RAM or Disk **BAD**
* Can put a hook in github which deploys on push.  Non-developers can use github directly to update content **GOOD**  
**Option: Push to Github for archiving, build the site locally, scp the built site to prod**  
  
**Option:  (Push to Github or Edit inside Github) , build on Jenkins, deploy with Jenkins**  
* Easy editing for Non-Developers
* Only 1 push required if using command line
* Jenkins can configure the Hook and the deploy destination
  
**Option:  (Push to GitHub or Edit inside GitHub) , build on Travis, create GitHub release on Travis,
call webhook on prod to trigger the webhook (using Travis)**
  
  
# Documenting the Travis Option:

First bring in the `.travis.yml`.  
```
language: ruby
cache: bundler
rvm:
- 2.4
before_script:
- chmod +x ./script/cibuild
script: "./script/cibuild"
branches:
  only:
  - master
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true
sudo: false
before_deploy:
- git config --local user.name "YOUR GIT USER NAME"
- git config --local user.email "YOUR GIT USER EMAIL"
- git tag "$(date +'%Y%m%d%H%M%S')-$(git log --format=%h -1)"
  
- echo -e "\n\n\nbefore_deploy\n\n\n"
- echo echoing the site directory
- ls /home/travis/build/MYACCOUNTNAME/MYREPONAME/_site/
  
deploy:
  provider: releases
  api_key:
    secure: CHANGEME1234123412341234123412341234
  file_glob: true
  file: /home/travis/build/MYACCOUNTNAME/MYREPONAME/_site.zip
  skip_cleanup: true
  script: echo -e "\n\n\ndeploy\n\n\n"
  on:
    repo: MYACCOUNTNAME/MYREPONAME
    branch: master
```

Just a couple things to change here.  The GitHub account name and repo name must be specified, and also the api_key.  
The only way I know right now to generate that key is to use the Travis CLI.
```
sudo apt install ruby ruby-dev
sudo gem install travis
```
Then generate the api_key  
```
travis encrypt {get the key from travis website}
```
# WTF travis, inconsistent flip floppy crap!
```
travis login --pro
travis encrypt -r myusername/myrepo MYTRAVISKEYFROMTRAVIS --pro
```
copy the key
  
# Blah I'm dumb.  This is what worked:  Generate a Personal Access Token in GitHub, use Travis to encrypt it

    
# OK, the interesting part:  Deploy a GitHub Release on your own Server
  
The `.travis.yml` tells Travis to build the Jekyll site after a push to GitHub, and also to deploy the built site to GitHub Releases.  
In GitHub, configure a webhook on the Releases event.  Set the webhook URL to your webserver, with some port that you pick.  
  
### Manually deploy the auto-deploy server
I'm using a kludgey poor man's version of other stuff, but I just want something to work ASAP!  
**In short, what I'm doing is setting up a Flask server to listen to a URL and whenever it is hit with a POST request, it calls a shell script.  That shell script fetches the latest release and deploys it into the web server root**  

# Comments on that flask deploy server
### Multiple Sites
The machine I'm hosting on is hosting multiple sites,
and each of them I want to deploy like described here.  
There are 2 options:  Have 2 auto-deploy servers, or
have 1 auto-deploy server that can take a parameter for which site to deploy.  
I didn't like having 2 because having an extra server seemed like overkill, and also, they'd both have to be on different ports.  
So I went with the 1 server, but the problem there is, it has to be parameterized.  2 Options there:  Use 2 different URL endpoints, or use 1 URL endpoint with a JSON payload containing the parameter.  
I chose to use 2 different URL endpoints because it was easy.  
But, when testing, it failed.  This is what happened.  
  
Again, here I am talking about 1 server that can deploy 2 sites:  X.com and Y.com.  For the actual web server that hosts the website, I use a name based HTTP proxy to achieve this (docker, nginx).  The deployer server is on this same machine.  The deployer server listens on port 5000 and the web servers are all behind the proxy on port 80.  In GitHub, the Push Webhook needs a URL.  X.com and Y.com have the same IP address because they are behind the proxy.  So I thought it'd be nice to put set the webhook URL to X.com/x or Y.com/y .  The URL endpoint is parameterized, and the domain name doesn't matter but is there for nice looks.  
Unfortunately, this didn't work due to SSL.  The deployer server is a HTTPS server.  The cert I generated is associated to a domain name, X.com.  When GitHub tried to hit Y.com, it didn't work because the domain name did not match the cert.  
Well, that sucks.  I set both webhooks to X.com.  I could have set them both to commondeployerZ.com.  Eh, whatever.  
  

