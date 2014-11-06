---
layout: post
title: "A Note about Private Class Members in CoffeeScript/JavaScript"
date: 2014-11-06 03:50:59 +0000
comments: true
draft: true
categories: 
  - IT
tags:
  - JavaScript
  - CoffeeScript
---
JavaScript is the programming language of the web. With the growth of the JavaScript engine [V8](https://code.google.com/p/v8/) (the google chrome browser JavaScript engine), which enable JavaScript to run faster, JavaScript is increasingly utilized on the server side, e.g. with [node.js](http://nodejs.org/), [phantomjs](http://phantomjs.org/), etc.

[CoffeeScript](http://coffeescript.org/) is an innovative language that compiles into JavaScript code. With CoffeeScript we can write our program logic much more faster with less and readable code.

> CoffeeScript is a little language that compiles into JavaScript. Underneath that awkward Java-esque patina, JavaScript has always had a gorgeous heart. CoffeeScript is an attempt to expose the good parts of JavaScript in a simple way.

Most mainstream programming languages adopt the Object-oriented Programming (OOP) paradigm, however, JavaScript does not support OOP natively. But we can still implement classes and objects in its own way. In this article we'll discuss about classes, objects and encapsulation in JavaScript/CoffeeScript.

<!--more-->

## Classes and Objects in JavaScript/CoffeeScript

In JavaScript, we use `function` and `prototype` to implement classes.

<pre>function Square(side) {
    this.side = side; // initialize class variable
}
Square.prototype.area = function() {
    return this.side * this.side;
}
var a = new Square(2);
console.log(a.area()); // output: 4
var b = new Square(3)
console.log(b.area()); // output: 9
</pre>

Usually we wrap the definition of a class or a block of code with function scope `(function() { /* class definition */ })()` mainly for the following considerations:

1. Let the browser load the function and execute it as a whole
2. Create a new scope to avoid creating/overriding fields in the global object `window`

In CoffeeScript, defining a class is much more simpler.

<pre>class Square
    constructor: (side) ->
        @side = side
    area: ->
        @side * @side
a = new Square(2)
console.log a.area() # output 4
b = new Square(3)
console.log b.area() # output 9
</pre>

Which compiles into:

<pre>var Square, a, b;
Square = (function() {
  function Square(side) {
    this.side = side;
  }
  Square.prototype.area = function() {
    return this.side * this.side;
  };
  return Square;
})();
a = new Square(2);
console.log(a.area());
b = new Square(3);
console.log(b.area());
</pre>

We can see that CoffeeScript wrap the definition of a class with function scope automatically. Actually, CoffeeScript uses a function scope to wrap every JavaScript file compiled from CoffeeScript in case of collision.

In class defined with JavaScript function and prototype, there is no private methods and variables. All class methods and class variables are public.

## Private Field Encapsulation

## Class Inheritance and Protected Fields

## Performance Issues

## Wrapping Up

## Further Reading
+ [Implementing Private and Protected Members in JavaScript](http://philipwalton.com/articles/implementing-private-and-protected-members-in-javascript/)
+ [Module Pattern in JavaScript and CoffeeScript](http://robots.thoughtbot.com/module-pattern-in-javascript-and-coffeescript)
+ [http://www.yuiblog.com/blog/2007/06/12/module-pattern/](http://www.yuiblog.com/blog/2007/06/12/module-pattern/)