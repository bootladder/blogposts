---
layout: post
title: Create React App, Take 2
---
# Preface
I got a react practice app going a month ago.  Tic-tac-toe demo.  Used create-react-app.  
It was running on the development server for a bit.  Then I shut it down, now I wanted to bring it back up.  
I just tried to run `npm start` or `npm run start` whatever it is, and it totally crashed my VPS.  
All of my open ssh sessions were getting these printed...
```
Message from syslogd@caseyjones at Mar 26 04:56:10 ...
 kernel:[445167.403797] BUG: soft lockup - CPU#0 stuck for 25s! [node:363]

Message from syslogd@caseyjones at Mar 26 04:56:36 ...
 kernel:[445193.778113] BUG: soft lockup - CPU#0 stuck for 25s! [postgres:414]

```
Woah.  Maybe I ran out of RAM?  That's a likely cause.  If not, that's crazy.  
  
# Anyway, I'll reboot with the admin console and do a fresh create-react-app
# Why is it taking so long... I don't remember it taking so long 
# Wow I didn't realize it was 200MB, fresh install.  It's all in node_modules

OK, now the dev server is up.  
My goal is to take some built JS and put it in my existing project.  
Let's just get straight to that.  run a `npm run build`.  
  
# What did npm run build do ?
First, `index.html`.  It took the source index.html, `public/index.html` , removed all the comments and white space, replaced the template `%PUBLIC_URL%` with `/`.  Ahh here's the cool part, at the end of the `<body>` , the following was inserted:  `<script type="text/javascript" src="/static/js/main.ee7b2412.js">`.  
And of course, our .js is in there.  
Cool, let's see if I can just pop that main.ee7b2412.js into my static site, and stick a couple HTML tags to load it up.  
So, in the HTML I need the following: `<div id="root"></div>`  
And at the end of the body I need this: `<script type="text/javascript" src="react-app.js"></script>`
# Cool, that works.  
But, can't use this for development, because it takes way too long to build the production site.  I'll have to go back to the development server and just start over there.
  
# OK I need another refresher... this stuff did not stick from last time.
* Everything in React is JS.  No HTML *
* What is a class?
* A class has a constructor that takes in `props` and sets initial `state`
* A class has a method `render()` that returns JSX markup.  JSX is the part that's like HTML.
* A class has life cycle methods like `componentDidMount()`
  
# I tried to copy paste some Bootstrap HTML as a starting point
Paste it into the `render()` function's return.
