# Prototype

## [[Prototype]]

Objects in JavaScript have an internal property, denoted in the specification as [[Prototype]], which is simply a reference to another object. Almost all objects are given a non-`null` value for this property, at the time of their creation.

when we reference a property on an object, such as `myObject.a` the default `[[Get]]` operations is invoked , the first step of it is to check if the property `a`  is present in `myObject` if its not present [[Get]] proceeds to follow the [[prototype]] link .

This process continues until either a matching property name is found, or the [[Prototype]] chain ends. If no matching property is ever found by the end of the chain, the return result from the [[Get]] operation is undefined.



```
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2
```

Similar to this [[Prototype]] chain look-up process, if you use a for..in loop to iterate over an object, any property that can be reached via its chain (and is also enumerable -- see Chapter 3) will be enumerated. If you use the in operator to test for the existence of a property on an object, in will check the entire chain of the object (regardless of enumerability).


```
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

for (var k in myObject) {
	console.log("found: " + k);
}
// found: a

("a" in myObject); // true

```

## Object.prototype

The top-end of every normal [[Prototype]] chain is the built-in Object.prototype. This object includes a variety of common utilities used all over JS, because all normal (built-in, not host-specific extension) objects in JavaScript "descend from" (aka, have at the top of their [[Prototype]] chain) the Object.prototype object.

## Setting and Shadowing property

consider

```
myObject.foo = "bar";
```

If the `myObject` object already has a normal data accessor property called `foo ` directly present on it, the assignment is as simple as changing the value of the existing property.

If `foo` is not already present directly on myObject, the [[Prototype]] chain is traversed, just like for the [[Get]] operation. If `foo` is not found anywhere in the chain, the property `foo` is added directly to `myObject` with the specified value, as expected.

However, if `foo` is already present somewhere higher in the chain, nuanced (and perhaps surprising) behavior can occur with the `myObject.foo = "bar"` assignment. 

We will now examine three scenarios for the myObject.foo = "bar" assignment when foo is not already on myObject directly, but is at a higher level of myObject's [[Prototype]] chain:

1. If a normal data accessor (see Chapter 3) property named `foo` is found anywhere higher on the [[Prototype]] chain, and it's not marked as read-only (writable:false) then a new property called `foo` is added directly to `myObject`, resulting in a shadowed property.
2. If a foo is found higher on the [[Prototype]] chain, but it's marked as read-only (writable:false), then both the setting of that existing property as well as the creation of the shadowed property on myObject are disallowed. If the code is running in strict mode, an error will be thrown. Otherwise, the setting of the property value will silently be ignored. Either way, no shadowing occurs.
3. If a foo is found higher on the [[Prototype]] chain and it's a setter, then the setter will always be called. No foo will be added to (aka, shadowed on) myObject, nor will the foo setter be redefined.

If you want to shadow `foo` in cases #2 and #3, you cannot use `=` assignment, but must instead use `Object.defineProperty(..)` to add foo to myObject.

## Class
In fact, JavaScript is almost unique among languages as perhaps the only language with the right to use the label "object oriented", because it's one of a very short list of languages where an object can be created directly, without a class at all.

In JavaScript, classes can't (being that they don't exist!) describe what an object can do. The object defines its own behavior directly. There's just the object.

## Class Functions

All functions by default get a public, non-enumerable property on them called prototype, which points at an otherwise arbitrary object.

```
function Foo() {
	// ...
}

Foo.prototype; // { }  
```
This object is often called "Foo's prototype", because we access it via an unfortunately-named `Foo.prototype` property reference. 

Each object created from calling new Foo() (see Chapter 2) will end up (somewhat arbitrarily) [[Prototype]]-linked to this "Foo dot prototype" object.

```
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

new Foo() results in a new object (we called it a), and that new object a is internally [[Prototype]] linked to the Foo.prototype object.

We end up with two objects, linked to each other. That's it. We didn't instantiate a class. We certainly didn't do any copying of behavior from a "class" into a concrete object. We just caused two objects to be linked to each other.

In fact, the secret, which eludes most JS developers, is that the new Foo() function calling had really almost nothing direct to do with the process of creating the link. It was sort of an accidental side-effect.

Can we get what we want in a more direct way? Yes! The hero is Object.create(..).

## What is in a name

In JavaScript, we don't make copies from one object ("class") to another ("instance"). We make links between objects. 

"Inheritance" implies a copy operation, and JavaScript doesn't copy object properties (natively, by default). Instead, JS creates a link between two objects, where one object can essentially delegate property/function access to another object. "Delegation" is a much more accurate term for JavaScript's object-linking mechanism.

## Constructor

Consider

```
function Foo() {
	// ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

The `Foo.prototype` object by default (at declaration time on line 1 of the snippet!) gets a public, non-enumerable (see Chapter 3) property called `.constructor`, and this property is a reference back to the function (`Foo` in this case) that the object is associated with. Moreover, we see that object a created by the "constructor" call `new Foo()` seems to also have a property on it called `.constructor` which similarly points to "the function which created it".

Note: This is not actually true. a has no `.constructor` property on it, and though `a.constructor` does in fact resolve to the `Foo` function, "constructor" does not actually mean "was constructed by", as it appears. In actuality, the .constructor reference is also delegated up to Foo.prototype, which happens to, by default, have a .constructor that points at Foo

## Constructor or call


Foo is no more a "constructor" than any other function in your program. Functions themselves are not constructors. However, when you put the new keyword in front of a normal function call, that makes that function call a "constructor call". In fact, new sort of hijacks any normal function and calls it in a fashion that constructs an object, in addition to whatever else it was going to do.

ex:

```
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

## Mechanics

Consider

```
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); // "a"
b.myName(); // "b"
```

This snippet shows two additional "class-orientation" tricks in play:

`this.name = name:` adds the `.name` property onto each object (`a and b`, respectively;), similar to how class instances encapsulate data values.

`Foo.prototype.myName = ...:` perhaps the more interesting technique, this adds a property (function) to the `Foo.prototype object`. Now, `a.myName()` works, but perhaps surprisingly. How?

In the above snippet, it's strongly tempting to think that when `a and b` are created, the properties/functions on the `Foo.prototype` object are copied over to each of `a and b` objects. However, that's not what happens.

At the beginning of this chapter, we explained the [[Prototype]] link, and how it provides the fall-back look-up steps if a property reference isn't found directly on an object, as part of the default [[Get]] algorithm.

So, by virtue of how they are created, `a and b` each end up with an internal [[Prototype]] linkage to `Foo.prototype`. When `myName` is not found on `a or b`, respectively, it's instead found (through delegation) on `Foo.prototype.`

Consider

```
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!

```

For one, the .constructor property on Foo.prototype is only there by default on the object created when Foo the function is declared. If you create a new object, and replace a function's default .prototype object reference, the new object will not by default magically get a .constructor on it.



## Prototypal Inheritance

<img src="img3.png">

Above figure which shows not only delegation from an object (aka, "instance") a1 to object Foo.prototype, but from Bar.prototype to Foo.prototype, which somewhat resembles the concept of Parent-Child class inheritance. Resembles, except of course for the direction of the arrows, which show these are delegation links rather than copy operations.

See below code prototype style code

```
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// here, we make a new `Bar.prototype`
// linked to `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

The important part is `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` creates a "new" object out of thin air, and links that new object's internal [[Prototype]] to the object you specify (`Foo.prototype` in this case).

In other words, that line says: "make a new 'Bar dot prototype' object that's linked to 'Foo dot prototype'."

When `function Bar() { .. }` is declared, `Bar`, like any other function, has a `.prototype` link to its default object. But that object is not linked to `Foo.prototype` like we want. So, we create a new object that is linked as we want, effectively throwing away the original incorrectly-linked object.

 Prior to ES6, there's a non-standard and not fully-cross-browser way, via the .`__proto__` property, which is settable. ES6 adds a `Object.setPrototypeOf(..) ` helper utility, which does the trick in a standard and predictable way.

```
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```








