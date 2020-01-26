---
layout: post
title: JuicyPixels Notes
date: 2019-05-16 19:58 -0400
---
Following this tutorial:  <https://www.stackbuilders.com/tutorials/haskell/image-processing/>
  
# Make rotateImg work on all image formats
The first example of working of code is `convertImg` which has the type `convertImg :: ImgFormat -> FilePath -> IO ()`.  It can read in an image of any format that can be read by the function `readImage :: FilePath -> IO (Either String DynamicImage)`, and then it converts that image to whatever image format.  
But the next example, `rotateImg`, only works on PNG images.  
  
  
Weird pattern matching syntax: 
```
rotateImg :: Image PixelRGB8 -> Image PixelRGB8
rotateImg img@Image {..} = runST $ do
```
What is `img@Image {..}`?  `img` is the name bound to the argument, `Image {..}` is pattern matching on the argument, `Image PixelRGB8` .  Usually you would see pattern matching eg. `myFunc (Just a) = id a` and here since `Image` is a record type, there are curly brackets.  `{..}` is an extension from `{-# LANGUAGE RecordWildCards #-}` saying the names bound to the fields are the names of the fields, eg `imageWidth`
