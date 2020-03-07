---
layout: post
title: React and AngularJS Notes
---
# React  
* A component is a self-contained module that renders some output
** For example, a button.  A component might include one or more other components in its output.  eg. a Form.
*  Wow, this caching thing on index.html is pretty terrible for a new user.
*  All components have a `render` function that SPECIFIES what the HTML output of the COMPONENT is.
*  JSX: JS Extension, allows writing JS that LOOKS LIKE HTML
```
class HelloWorld extends React.Component {
  render() {  //The component has a RENDER Function
    return (  //The render function RETURNS HTML
      <h1 className='large'>Hello World</h1>  //THIS IS NOT ACTUALLY HTML
    );
  }
}
```
* h1 className GETS CONVERTED TO class.  This is because CLASS IS A RESERVED JS KEYWORD.
* JSX is very convenient notation for writing nested HTML snippets
* In order to load React from <script></script> , ie. client side, the following is needeed:
* react.min.js , react-dom.min.js, babel-core@x.x.x/browser.min.js
* Babel is a library for transpiling ES6 to ES5.
* Inside the body we have the following:
```
<script type="text/babel">
var app = <h1>Hello world</h1>
var mountComponent = document.querySelector('#app');
ReactDOM.render(app, mountComponent);
</script>
``` 
* This tells Bable to convert the above ES6 into ES5
* render() :  `ReactDOM.render(<what>, <where>)`
* Interesting, in this case the WHERE is #app.  In other tutorials it was document root.
* In the above snippet, app is a HTML snippet, directly passed into render().
* Another way to do it is this:  `ReactDOM.render(<App />, mount);` Where App is a class like class HelloWorld
* Ah, this makes sense why I struggled with the meta tags.  They go inside head.  In
   these examples, the rendering goes to document root or a #hash section.
* As a test, I should be able to call render() twice in a row and get 2 components to render.
** Cool.  After doing that I saw that, if render() is called twice in a row on the same DOM mount (what's it called?)
  This is intuitive, the DOM mount gets overwritten.  Using 2 div tags with different classes, render() can be called on each of them.
* A "Wrapper" component combines multiple components.  The Wrapper will have divs with classnames.  
  The Wrapper will nest other divs inside those divs, with classnames.  Those child components 
  can be used in the Parent by tag notation.
```
class App extends React.Component {
  render() {
    return (
      <div className="notificationsFrame">
        <div className="panel">
          <Header />
          <Content />
        </div>
      </div>
    )
  }
}
```
# To add data ie. props to a component
* Write something like this:  `<Header title="My Header">` It is picked up by the Component liek this:  
```
<span className="title">
  {this.props.title}
</span>
```
# https://reactjs.org/tutorial/tutorial.html was informative
# https://reactjs.org/docs/state-and-lifecycle.html
* I need to learn how setting state works.
* Do I need a constructor to have this.state?
* Is this.setState() the only way to set state? (except initializing state in constructor?)
* This is what I'm doing:
** In render() I am using {this.state.mystatevar}
** In componentDidMount() I am using this.setState{mystatevar:myval};
** But the console says error: cannot read property mystatevar of null.
** It appears the answer is Yes, you have to declare the different state variables in the constructor

# Uggh, I don't know how to separate the JS into different .js files.
# All I want is a index.html and a index.js












# AngularJS
First you have this:  `<html ng-app="todoApp">`  
And eventually this:  `<div ng-controller="TodoListController as todoList">`
Then in the .js you have:  
```
angular.module('todoApp', [])
  .controller('TodoListController', function() {
```
The names have to match.  
`<html ng-app="todoApp">`  Does not have to be at the top, or in a html tag.  
In order to use $http requests, the controller is passed $http.  As in this:  
```
angular.module('myapp', [])
  .controller('myController', function($http) {
```
$http will give `No 'Access-Control-Allow-Origin' header is present on the requested resource.'`

*Meh, I don't like it.*  
*As soon as I found out how much has changed over the versions I immediately quit.  I'm not going to bother learning the history when I already think it sucks*
