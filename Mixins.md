# Mixins

## Javascript Classes

JS has had some class-like syntactic elements (like new and instanceof) for quite awhile, and more recently in ES6, some additions, like the class keyword.

Since classes are a design pattern, you can, with quite a bit of effort (as we'll see throughout the rest of this chapter), implement approximations for much of classical class functionality. JS tries to satisfy the extremely pervasive desire to design with classes by providing seemingly class-like syntax.

A class is a blue-print. To actually get an object we can interact with, we must build (aka, "instantiate") something from the class. The end result of such "construction" is an object, typically called an "instance", which we can directly call methods on and access any public data properties from, as necessary.

## Constructor

Instances of classes are constructed by a special method of the class, usually of the same name as the class, called a constructor. This method's explicit job is to initialize any information (state) the instance will need.

```
class CoolGuy {
	specialTrick = nothing

	CoolGuy( trick ) {
		specialTrick = trick
	}

	showOff() {
		output( "Here's my trick: ", specialTrick )
	}
}

Joe = new CoolGuy( "jumping rope" )

Joe.showOff() // Here's my trick: jumping rope
```

## Class Inheritance

```
class Vehicle {
	engines = 1

	ignition() {
		output( "Turning on my engine." )
	}

	drive() {
		ignition()
		output( "Steering and moving forward!" )
	}
}

class Car inherits Vehicle {
	wheels = 4

	drive() {
		inherited:drive()
		output( "Rolling on all ", wheels, " wheels!" )
	}
}

class SpeedBoat inherits Vehicle {
	engines = 2

	ignition() {
		output( "Turning on my ", engines, " engines." )
	}

	pilot() {
		inherited:drive()
		output( "Speeding through the water with ease!" )
	}
}
```

Note: For clarity and brevity, constructors for these classes have been omitted.

We define the `Vehicle` class to assume an engine, a way to turn on the ignition, and a way to drive around. But you wouldn't ever manufacture just a generic "vehicle", so it's really just an abstract concept at this point.

So then we define two specific kinds of vehicle: `Car` and `SpeedBoat`. They each inherit the general characteristics of `Vehicle`, but then they specialize the characteristics appropriately for each kind. A `car` needs 4 wheels, and a speed boat needs 2 engines, which means it needs extra attention to turn on the ignition of both engines.

## Polymorphism

Car defines its own drive() method, which overrides the method of the same name it inherited from Vehicle. But then, Cars drive() method calls inherited:drive(), which indicates that Car can reference the original pre-overridden drive() it inherited. SpeedBoats pilot() method also makes a reference to its inherited copy of drive().

This technique is called "polymorphism", or "virtual polymorphism". More specifically to our current point, we'll call it "relative polymorphism".

## Multiple Inheritance

Some class-oriented languages allow you to specify more than one "parent" class to "inherit" from. Multiple-inheritance means that each parent class definition is copied into the child class.

On the surface, this seems like a powerful addition to class-orientation, giving us the ability to compose more functionality together. However, there are certainly some complicating questions that arise. If both parent classes provide a method called drive(), which version would a drive() reference in the child resolve to? Would you always have to manually specify which parent's drive() you meant, thus losing some of the gracefulness of polymorphic inheritance?

There's another variation, the so called "Diamond Problem", which refers to the scenario where a child class "D" inherits from two parent classes ("B" and "C"), and each of those in turn inherits from a common "A" parent. If "A" provides a method drive(), and both "B" and "C" override (polymorph) that method, when D references drive(), which version should it use (B:drive() or C:drive())?

JavaScript is simpler: it does not provide a native mechanism for "multiple inheritance". Many see this as a good thing, because the complexity savings more than make up for the "reduced" functionality. But this doesn't stop developers from trying to fake it in various ways, as we'll see next.

## Mixins

JavaScript's object mechanism does not automatically perform copy behavior when you "inherit" or "instantiate". Plainly, there are no "classes" in JavaScript to instantiate, only objects. And objects don't get copied to other objects, they get linked together

 JS developers fake the missing copy behavior of classes in JavaScript: mixins. 
 Two types of "mixin": 
 1. Explicit
 2. Implicit.

 ### Explicit Mixins

 Below is utility thant manuall copies behaviour from one javascript object to another. Libraries call this `extend`, For illustration we can call this as `Mixin`

 ```
 // vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}

var Vehicle = {
	engines: 1,

	ignition: function() {
		console.log( "Turning on my engine." );
	},

	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,

	drive: function() {
		Vehicle.drive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	}
} );
 ```

 Let's examine this statement: `Vehicle.drive.call( this )`. This is what I call "explicit pseudo-polymorphism". Recall in our previous pseudo-code this line was `inherited:drive()`, which we called "relative polymorphism".

JavaScript does not have (prior to ES6; see Appendix A) a facility for relative polymorphism. So, because both `Car` and `Vehicle` had a function of the same `name: drive()`, to distinguish a call to one or the other, we must make an absolute (not relative) reference. We explicitly specify the `Vehicle` object by name, and call the `drive()` function on it.

But if we said `Vehicle.drive()`, the this binding for that function call would be the `Vehicle` object instead of the `Car` object (see Chapter 2), which is not what we want. So, instead we use `.call( this )` (Chapter 2) to ensure that `drive()` is executed in the context of the `Car` object.

But because of JavaScript's peculiarities, explicit pseudo-polymorphism (because of shadowing!) creates brittle manual/explicit linkage in every single function where you need such a (pseudo-)polymorphic reference. This can significantly increase the maintenance cost. Moreover, while explicit pseudo-polymorphism can emulate the behavior of "multiple inheritance", it only increases the complexity and brittleness.

The result of such approaches is usually more complex, harder-to-read, and harder-to-maintain code. Explicit pseudo-polymorphism should be avoided wherever possible, because the cost outweighs the benefit in most respects.

Since the two objects also share references to their common functions, that means that even manual copying of functions (aka, mixins) from one object to another doesn't actually emulate the real duplication from class to instance that occurs in class-oriented languages.

JavaScript functions can't really be duplicated (in a standard, reliable way), so what you end up with instead is a duplicated reference to the same shared function object (functions are objects; see Chapter 3). If you modified one of the shared function objects (like ignition()) by adding properties on top of it, for instance, both Vehicle and Car would be "affected" via the shared reference.

Take care only to use explicit mixins where it actually helps make more readable code, and avoid the pattern if you find it making code that's harder to trace, or if you find it creates unnecessary or unwieldy dependencies between objects.

### Implicit Mixins

```
var Something = {
	cool: function() {
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
	cool: function() {
		// implicit mixin of `Something` to `Another`
		Something.cool.call( this );
	}
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (not shared state with `Something`)
```

With Something.cool.call( this ), which can happen either in a "constructor" call (most common) or in a method call (shown here), we essentially "borrow" the function Something.cool() and call it in the context of Another (via its this binding; see Chapter 2) instead of Something. The end result is that the assignments that Something.cool() makes are applied against the Another object rather than the Something object.

So, it is said that we "mixed in" Somethings behavior with (or into) Another.

While this sort of technique seems to take useful advantage of this rebinding functionality, it is the brittle Something.cool.call( this ) call, which cannot be made into a relative (and thus more flexible) reference, that you should heed with caution. Generally, avoid such constructs where possible to keep cleaner and more maintainable code.


