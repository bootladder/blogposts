---
layout: post
title: Porting This Blog to freejekyllbuilder.com
---
I currently had some other `.travis.yml` and `cibuild` script.  
I should be able to just replace those files,
edit cibuild to hit the deployer webhook.  
Probably something won't be installed?

  
appears that the jekyll/jekyll image
on Docker Hub does work.
  
try again?
  
# The Pipeline
Push a post to github.  Repo contains the jekyll source and the posts in `_posts/`.  
Github triggers a travis build.  
`.travis.yml` executes a script, `cibuild`, which zips the site source and
`curl`s it to <freejekyllbuilder.com>.  The curl request returns, and then
a webhook on prod is triggered.  
Prod has the URL for the download already, so it just grabs the site, unzips and deploys
  
# Gets confusing... slack hooks would be cool
