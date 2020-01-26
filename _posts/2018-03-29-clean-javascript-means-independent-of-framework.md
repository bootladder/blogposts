---
layout: post
title: Clean Javascript means Independent of Framework
---
I have no interest in climbing steep learning curves for all these javascript frameworks.  Also I don't want to learn stuff that will be obsolete soon.  
That's why I like C programming and embedded systems.  Nothing has really changed.  You learn it once and you're good to go.  
Somewhere in the middle of these 2 is Clean Architecture, or just modularity.  
  
I feel like React kind of throws a wrench into this.  Let me talk it out to myself.  
JSX is not HTML.  It can be transformed into HTML, but it's not HTML.  It would be nice if it was, so HTML could be reusable.  
But we can ignore this and leave that hassle for the front end person.  
I should be able to prototype my UI without any CSS, and the HTML should be throwaway code.  
  
It's frustrating trying to take what I want, or what I already have, and make it work with React.  So, let's not do that.  What if all the React code looked exactly like the examples on the web?  That solves 2 problems:  first it takes away the pain of working with the framework, second it keeps my "core logic" separated.  
  
## What are some of the essential elements of React.  Why am I even using it?
The component class:  
A component has state, props, lifecycle events, and a render.  
  
When the state changes, the component rerenders.  
Pieces of state can be passed to children as props.  

# Interesting:  Porting Vanilla JS to React !!!
I have a vanilla JS function that creates some elements using `document.createElement()`, sets some to be children, populates some innerHTML, then returns the top parent.  
How do I port this same function to React?  Well.... looks like I can't.  
Take a look at the vanilla version here:
```
function createMessageListEntryFromMessageDesc(md) {

    var d = document.createElement('div')
    var playbutton = document.createElement('button')

    // Set innerHTML of Button
    playbutton.innerHTML = md.topic
    if( md.customtopic ) {
        playbutton.innerHTML = md.customtopic
    }

    // Set Color of Button
    var color
    if( md.project == "1" ) {
      color = "blue"
    }
    else {
      color = "gray"
    }

    // Set Opacity of Button
    var opacity
    if( md.listenedto == false )
        opacity = "1"
    else
        opacity = "0.2"

    playbutton.setAttribute("style",
      "background-color: "+color+";font-size : 32px; opacity:"+opacity);

    d.appendChild(playbutton)
    return d
}
```
  
Pretty straight forward.  It uses `document.createElement()` , sets properties `innerHTML` and calls method `setAttribute()` and `appendChild()`.  
  
Now see what I did to make it work with React.  (without JSX)
```
function testerdom(md) {

    var text = md.topic
    if( md.customtopic ) {
        text = md.customtopic
    }
    // Set Color of Button
    var color
    if( md.project == "1" ) {
      color = "blue"
    }
    else {
      color = "gray"
    }

    // Set Opacity of Button
    var opacity
    if( md.listenedto == false )
        opacity = "1"
    else
        opacity = "0.2"

    var props = {
      style:{
        backgroundColor:color,
        fontSize: '32px',
        opacity:opacity
      }
    }
    var playbutton = React.createElement('button',props,text)
    var d = React.createElement('div',null,playbutton)
    return d
}
```
  
So, actually in terms of lines of code it isn't horrible.  Those if statements are only setting plain old variables, so I can just factor those out.  The lame part is that `React.createElement()` has a totally different API from `document.createElement()`.  The props, not only have to be converted to object like that, but it's super annoying that the fields changed, eg. background-color changed to backgroundColor.  Being that I want to just do the bare minimum CSS, this is annoying.  
1 more note, to set children of a node in react, you pass them into the parent when the parent is being created.  Notice the top level node, the `div`, is created on the last line.  This is because React nodes are immutable.  In Plain JS, I could just `appendChild()`
  
## Conversely, what is lacking from Vanilla JS?

# No Component Classes
What does a component class ala React do for me?  
Lifecycle methods are cool.  With Vanilla JS, all I had was button onclick.  

Let's do an experiment and mimic some of React's behavior.  
I want a HTML snippet like this
```
<body>
<div id=root />
<script src="my-vanilla.js"/>
</body>
```
The javascript executes on load.  It will render my "App Component" under root.
*Phew, fortunately there are no ' characters in there*
  
Crap.  I can load my HTML from a file into the div, using jQuery, but, I need to ensure that the (JS that sets the button onClick handlers) is executed after the div is loaded.  Otherwise javascript says the element is null.  
  
OK, got out of that using the callback function on jQuery's `load`.  After the HTML is loaded, the callback executes JS which will assign the click handlers.
  
This is not what I intend to do, this is just a refactor/saving of my work before I write different stuff.  
I'd like to have components ala React.  To do that, I need to dynamically generate the HTML, not just load static HTML into a #div.  Well not necessarily; actually, inside the static HTML, I could put in stub IDs in the divs, which could be set to application specific stuff, if the surrounding tag is identified.
  
To summarize.  What I did was take all the (JS that sets the button onclick handlers) , wrap all those into a function.  That function is called in the callback of jquery load.  Another thing I did was bring the declarations of those DOM object variables to the global scope, and initialized them in the wrapper function.  This gives other functions access to those global DOM objects.  Note the DOM objects were previously global.  They became function scope when I wrapped them into a function.  So I brought out only the declarations of their names to the global scope.
  
  
# Thinking out loud, how to Component-ize my widget?
My widget displays a list of messages between two people.  
Really, the only things that need to be identified are those 2 people.  
  
Let's list out actually what's there:  
* Button to load messages  
* Array of messages that are loaded
* 2 Divs to hold each displayed message, one A-to-B and another B-to-A
* AJAX call, which POSTs a form with the 2 people
* AJAX callback, takes the result, passes to a function that renders
* Rendering

The rendering needs to create DOM elements that have IDs in them, and onclick handlers that can access those IDs.  For example to delete a message, the ID of the message has to be passed to a function when a button is clicked.
  
# Obvious Realization #1:
## To have multiple instances of the same thing, you need classes.
## Actually wait, maybe we can just use objects?
As with any other language, class will give you stuff like constructors, methods that can use `this`, 

# Embarassing Realization #2:
https://blog.angular-university.io/really-understanding-javascript-closures/
Just.... read this.
