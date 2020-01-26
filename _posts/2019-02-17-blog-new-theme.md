---
layout: post
title: 'Jekyll Blog: Mixing Themes for About, Home, Page'
---
Jekyll themes don't mix together.  There's a `main.css` that gets built from a `main.scss` , and that is referenced by `head.html`.  All the themes I've seen have the same naming convention, so they conflict.  
  
My plan is to have my About, Home and Page pages all have different themes.  To do this, I will bring in the 3 themes into my project and rename all the conflicting names for the 3 themes.

