---
layout: post
title: Deploy blog updates automatically with git push
categories:
  - "Ops"
---

After I had the docker shared volume + git solution working (see other post),
I immediately noticed there were too many steps to work on the blog.
  
Change blog source --> jekyll build --> git add && git commit && git push --> git pull
  
The worst part being that git pull happens on the production host and the others happen on dev host.
  
So, I found a bunch of posts talking about a push-to-deploy solution using git --bare.
It works by pushing commits to a bare repository located on production host.
The bare repo then has a post receive hook which installs files somewhere else on the production filesytem.
  
I want to skip a step there, because fortunately, the Docker setup for this blog
maps the web server root directory to inside the git repo.  So no separate install directory needed.
  
This guide is relevant, actually obvious but it's nice to see it in a guide.  https://gist.github.com/joahking/780877
  
Trying it out now...
  
* Stop the running Docker container
* Go into a new directory so we don't touch the working install
* git clone --bare https://myrepo
* I get a blog.git directory, no source.
  Ahh, now I get it.  There's no working tree, there's only objects (blobs).  The raw git stuff.
  A working tree at any commit can be constructed using the objects and the history (did I say that right?).
  
* So, I can't just have a bare repo.  I can't have a non-bare repo because I can't push to it.
* So I guess the other guys were right.  
  Push to the bare repo and use the post receive hook
  to create a working copy and install that.
* But that doesn't work if my Docker container is already running
* If Docker container is running, that means docker-compose up happened inside a git repo. (in my case)
  If there was a separate bare repo, its post receive hook would install files inside another git repo.
  Which is not too bad since I won't ever push from that repo.
  Now consider if the Dockerfile changes.  The container has to be rebuilt and restarted.
  Pulling into the production non-bare repo might be awkward since its data is behind master in the history.
  The post receive hook could copy the Dockerfile.
  
* This may be more complicated than I like.  But perhaps the proper solution is best.
  On production host we have the bare repo.  
  On dev we push to production -->  bare repo executes post receive
  That post receive will install the Dockerfile, docker-compose.yaml, and the static website
  into another location.  
  Actually this is nice since docker-compose.yaml uses relative paths "./" when mapping the
  volume for http server root.  
  So we can actually deploy the website in any directory.
  
* Nice! It worked as expected.
* git remote add prod
* git push prod master
* the bare prod repo executes its script and files appear inside the deploy directory!

I used the post-receive script from here
https://gist.github.com/thomasfr/9691385
  
Works great.
  
Actually you know what's really cool, I don't even need github.com to do this!  It's just my laptop and prod.
  
I could actually automate this further and combine:  jekyll build --> git add . --> git commit -m "insert message here" --> git push 
Into 1 command.

# Now I can deploy changes to my blog with 1 git push!  As demonstrated by this particular change!
  
