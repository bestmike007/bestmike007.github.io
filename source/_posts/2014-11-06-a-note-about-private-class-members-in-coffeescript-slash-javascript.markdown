---
layout: post
title: "A Note about Private Class Members in CoffeeScript/JavaScript"
date: 2014-11-06 03:50:59 +0000
comments: true
categories: 
  - IT
tags:
  - Private Class Members
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

<pre><code class="language-javascript">function Square(side) {
    this.side = side; // initialize class variable
}
Square.prototype.area = function() {
    return this.side * this.side;
}
var a = new Square(2);
console.log(a.area()); // output: 4
var b = new Square(3)
console.log(b.area()); // output: 9
</code></pre>

Usually we wrap the definition of a class or a block of code with function scope `(function() { /* class definition */ })()` mainly for the following considerations:

1. Let the browser load the function and execute it as a whole
2. Create a new scope to avoid creating/overriding fields in the global object `window`

In CoffeeScript, defining a class is much more simpler.

<pre><code class="language-coffeescript">class Square
    constructor: (side) ->
        @side = side
    area: ->
        @side * @side
a = new Square(2)
console.log a.area() # output 4
b = new Square(3)
console.log b.area() # output 9
</code></pre>

Which compiles into:

<pre><code class="language-javascript">var Square, a, b;
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
</code></pre>

We can see that CoffeeScript wrap the definition of a class with function scope automatically. Actually, CoffeeScript uses a function scope to wrap every JavaScript file compiled from CoffeeScript in case of collision.

In classes defined with JavaScript function and prototype, there is no private methods and variables. All class methods and class variables are public.

## Private Field Encapsulation

The basic thought of implementing private field is to encapsulate variables and methods in anonymous function scope. The following example adds private static variables and methods in a class.

<pre><code class="language-coffeescript"># CoffeeScript
class Counter
    # private static variable
    counter = 0
    # private static method
    countInstance = ->
        counter++
    # public static method
    @instanceCount = ->
        counter
    constructor: ->
        countInstance()
c1 = new Counter()
c2 = new Counter()
console.log Counter.instanceCount() # output 2
</code></pre>

Which compiles into:

<pre><code class="language-javascript">var Counter, c1, c2;
Counter = (function() {
  var countInstance, counter;
  counter = 0;
  countInstance = function() {
    return counter++;
  };
  Counter.instanceCount = function() {
    return counter;
  };
  function Counter() {
    countInstance();
  }
  return Counter;
})();
c1 = new Counter();
c2 = new Counter();
console.log(Counter.instanceCount());
</code></pre>

The first function scope to create the class Counter is created only once; so that the variables and methods inside has only one instance and they are accessible from all class methods and instance methods. In order to implement private instance fields, we have to create a new function scope every time the class is instantiated. Considering that all private instance fields have to be accessible from all instance methods, public methods must be either inside the same function scope of the private fields or in one of the descendants, which means that we have to define public methods every time a instance is created so that prototype is not suitable for such situation. Here is an example of class definition with public and private fields and static fields.

<pre><code class="language-coffeescript"># CoffeeScript
class Square
    # private static variable
    counter = 0
    # private static method
    countInstance = ->
        counter++; return
    # public static method
    @instanceCount = ->
        counter
    constructor: (side) ->
        countInstance()
        # side is already a private variable, we define a private variable `self` to avoid evil `this`
        self = this
        # private method
        logChange = ->
            console.log "Side is set to #{side}"
        # public methods
        self.setSide = (v) ->
            side = v
            logChange()
        self.area = ->
            side * side
s1 = new Square(2)
console.log s1.area()   # output 4
s2 = new Square(3)
console.log s2.area()   # output 9
s2.setSide 4            # output Side is set to 4
console.log s2.area()   # output 16
console.log Square.instanceCount() # output 2
</code></pre>

Which compiles into:

<pre><code class="language-javascript">var Square, s1, s2;
Square = (function() {
  var countInstance, counter;
  counter = 0;
  countInstance = function() {
    counter++;
  };
  Square.instanceCount = function() {
    return counter;
  };
  function Square(side) {
    var logChange, self;
    countInstance();
    self = this;
    logChange = function() {
      return console.log("Side is set to " + side);
    };
    self.setSide = function(v) {
      side = v;
      return logChange();
    };
    self.area = function() {
      return side * side;
    };
  }
  return Square;
})();
s1 = new Square(2);
console.log(s1.area());
s2 = new Square(3);
console.log(s2.area());
s2.setSide(4);
console.log(s2.area());
console.log(Square.instanceCount());
</code></pre>

That's a way to implement class member encapsulation in normal cases (not too many instances and do not need inheritance). What about protected members and class inheritance?

## Class Inheritance and Protected Fields

Without encapsulated fields, class inheritance is easy to implement with function and prototype. And that's how [protoype.js](http://prototypejs.org/) implement class inheritance. For example:

<pre><code class="language-coffeescript"># CoffeeScript run with prototype.js
Rectangle = Class.create {
    initialize: (a, b) ->
        this.a = a
        this.b = b
    area: ->
        this.a * this.b
}
r = new Rectangle(2, 3)
console.log r.area() # output 6

Square = Class.create Rectangle, {
    initialize: ($super, side) ->
        $super side, side
}
s = new Square(4)
console.log s.area() # output 16
</code></pre>

Which compiles into:

<pre><code class="language-javascript">var Rectangle, Square, r, s;

Rectangle = Class.create({
  initialize: function(a, b) {
    this.a = a;
    return this.b = b;
  },
  area: function() {
    return this.a * this.b;
  }
});
r = new Rectangle(2, 3);
console.log(r.area());

Square = Class.create(Rectangle, {
  initialize: function($super, side) {
    return $super(side, side);
  }
});
s = new Square(4);
console.log(s.area());</code></pre>

To learn more about the prototype.js implementation, please read [the source code](https://github.com/sstephenson/prototype/blob/master/src/prototype/lang/class.js).

We've discussed about private members in JavaScript classes above. What about inheritance without prototype and protected fields accessible from sub-classes?

To create a subclass B from class A, we have to create a scope chain like this:

<pre>
A_private_static_scope <--- A_protected_scope <--- A_private_scope <--- A_public_scope
                                    ^
                                    |
B_private_static_scope <--- B_protected_scope <--- B_private_scope <--- B_public_scope
</pre>

This is theoretically impossible. Reasons:

1. Scope `B_protected_scope` cannot be in `A_protected_scope` AND `B_private_static_scope`
2. Even if we do not need static members, B is not defined inside the definition of class A (of course subclass is defined somewhere else), so that `B_protected_scope` cannot be in `A_protected_scope`
3. Methods in `B_protected_scope` should have access to members in `A_public_scope` and have no access to members in `A_private_scope`, which is impossible with scope chain
 
Conclusion is: we cannot implement class inheritance and encapsulation with JavaScript scope chain. However, we can implement our own class helper to implement class inheritance and protected members. I've created a lightweight Class Helper example to explain the means by which to implement private/protected/public static/instance variables/methods and subclass inheritance.

<pre><code class='language-coffeescript'>(->
	classChain = []

	getSuperClass = (subClass) ->
		return null if !subClass
		for cls in classChain
			return cls.superClass if cls.cls == subClass

	getInitializer = (_cls) ->
		return null if !_cls
		for cls in classChain
			return cls.initialize if cls.cls == _cls
		
	window.createClass = (superClass, classCreator) ->
		(classCreator = superClass; superClass = null) if arguments.length == 1
		return null if typeof classCreator != 'function'
		$parent = superClass
		$parents = []
		$parents.push superClass if superClass
		while getSuperClass($parent)
			$parent = getSuperClass($parent)
			$parents.push $parent
		cls = ->
			self = this
			$self = {}
			classInitializers = []
			originalInitializers = []
			for $parent in $parents
				originalInitializers.unshift getInitializer($parent)
			originalInitializers.push initialize
			for _initializer in originalInitializers
				((_initializer) ->
					classInitializers.push(->
						$self.initialize = classInitializers.pop() || (->)
						args = [self, $self]
						for arg in arguments
							args.push arg
						_initializer.apply self, args
						delete $self.initialize
						return if _initializer == initialize
						$super = {}
						$super.$super = $self.$super if $self.$super
						for k, v of $self
							$super[k] = v if typeof v == 'function'
						$self.$super = $super
					)
				)(_initializer)
			classInitializers.pop().apply(self, arguments)
			return
		initialize = classCreator(cls)
		classChain.push {
			cls: cls
			superClass: superClass
			initialize: initialize
		}
		return cls
)()</code></pre>

With the class creator helper method, we can create our own classes. For example:

<pre><code class='language-coffeescript'>A = createClass((Class)->
	# private static variable
	counter = 0
	# private static method
	countInstance = ->
		counter++; return
	# public static method
	Class.instanceCount = ->
		counter
	# class initiation method, where:
	# self: the instance object
	# $self: the context for protected members
	(self, $self, id) ->
		$self.initialize() # call super class constructor.
		countInstance()
		# private variable: id
		# private method
		getClassName = ->
			"A"
		# protected variables
		$self.id = id
		# protected method
		$self.present = ->
			"#{getClassName()} # #{$self.id}"
		# public methods
		self.print = ->
			console.log "<#{$self.present()}>"
)

B = createClass(A, (Class) ->
	counter = 0
	countInstance = ->
		counter++; return
	Class.instanceCount = ->
		counter
	(self, $self, id) ->
		$self.initialize(id) # call super class constructor.
		countInstance()
		getClassName = ->
			"B"
		# override
		$self.present = ->
			"#{getClassName()} : #{$self.$super.present()}"
)

a = new A('001')
b = new B('002')
a.print()   # output: "&lt;A # 001&gt;" 
b.print()   # output: "&lt;B : A # 002&gt;" 
console.log A.instanceCount() # output: 2
console.log B.instanceCount() # output: 1
</code></pre>

Protected members in this example are not strictly 'protected', but are 'protected abstract' and can be override in subclass. That's because protected methods are not implemented with scope chain in this class helper. And there might be better implementation. If you don't understand the example listed above, maybe you should learn more about JavaScript scope chain, prototype and closure.

Fortunately, in most cases we do not need protected members, at least we usually do not need protected methods that cannot be override. So that the class helper like above is sufficient for most use cases. Please note that the class helper above is not well tested for production use.

## Performance Issues

To encapsulate private members, we create public methods every time a class is initiated, so that all private members are accessible to public methods. That's how we protect the 'privacy'. Price is that it causes performance issues when thousands of instances are being created in our application. In such cases we fall back to the function & prototype scheme which does not implement private members, which however, is able to avoid defining public methods repeatedly.

In fact, 'privacy' is not strictly 'privacy' in JavaScript; with the modern tools such as the development tool in google chrome, we can inspect almost everything at ease. The 'privacy' here actually refers to the scheme to prevent unexpected access to encapsulated variables and methods. With that in mind, a proper naming convention to distinguish private and public members is sufficient, though everything is 'public'.

However, if you're still interested in the class helper in this article, please let me know.

## Further Reading

+ [Private Members in JavaScript](http://javascript.crockford.com/private.html)
+ [Implementing Private and Protected Members in JavaScript](http://philipwalton.com/articles/implementing-private-and-protected-members-in-javascript/)
+ [Module Pattern in JavaScript and CoffeeScript](http://robots.thoughtbot.com/module-pattern-in-javascript-and-coffeescript)
+ [A JavaScript Module Pattern](http://www.yuiblog.com/blog/2007/06/12/module-pattern/)