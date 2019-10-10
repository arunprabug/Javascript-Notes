# Objects

## Forms

1. Literal form

```
var myObj = {
	key: value
	// ...
};
```

2. Constructed form

```
var myObj = new Object();
myObj.key = value;
```

## Type
Objects are one of the 6 primary types. (string,number,boolean,null, undefined, object)

`function` is a sub-type of object (technically, a "callable object"). Functions in JS are said to be "first class" in that they are basically just normal objects (with callable behavior semantics bolted on), and so they can be handled like any other plain object.

`Array` are also form of objects with extra behaviour.

### Built in object
String, Number, Boolean, Object, Function, Array, Date, RegExp, Error

In JS, these are actually just built-in functions. Each of these built-in functions can be used as a constructor (that is, a function call with the new operator), with the result being a newly constructed object of the sub-type in question. 

## Contents

The contents of an object consist of values (any type) stored at specifically named locations, which we call properties.

It's important to note that while we say "contents" which implies that these values are actually stored inside the object, that's merely an appearance. The engine stores values in implementation-dependent ways, and may very well not store them in some object container. What is stored in the container are these property names, which act as pointers (technically, references) to where the values are stored.

See below

```
var myObject = {
	a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

The functions assigned to Javascript object property is not a method unlike other OO languages.

Arrays are objects, so even though each index is a positive integer, you can also add properties onto the array:

```
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length;	// 3

myArray.baz;	// "baz"
```

## Duplicating objects

1. Shallow copy
2. Deep copy

### Different ways

```
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

```
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

## Property Descriptors

Prior to ES5, the JavaScript language gave no direct way for your code to inspect or draw any distinction between the characteristics of properties, such as whether the property was read-only or not.

Get property descriptors of an object

```
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

## Define a new property with descriptors 

```
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2`
```

### Writable

The ability for you to change the value of a property is controlled by `writable`.

```
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```
As you can see, our modification of the value silently failed. If we try in strict mode, we get an error:


### Configurable

As long as a property is currently configurable, we can modify its descriptor definition, using the same `defineProperty(..)` utility.

```
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// not configurable!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

The final defineProperty(..) call results in a TypeError, regardless of strict mode, if you attempt to change the descriptor definition of a non-configurable property. Be careful: as you can see, changing configurable to false is a one-way action, and cannot be undone!

**Note:** There's a nuanced exception to be aware of: even if the property is already configurable:false, writable can always be changed from true to false without error, but not back to true if already false.

Another thing `configurable:false` prevents is the ability to use the delete operator to remove an existing property.

### Enumerable

All normal user-defined properties are defaulted to enumerable, as this is most commonly what you want. But if you have a special property you want to hide from enumeration(`for...in`), set it to `enumerable:false.`

### Immutability

It is sometimes desired to make properties or objects that cannot be changed (either by accident or intentionally). ES5 adds support for handling that in a variety of different nuanced ways.

It's important to note that all of these approaches create shallow immutability. That is, they affect only the object and its direct property characteristics. If an object has a reference to another object (array, object, function, etc), the contents of that object are not affected, and remain mutable.

```
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

### Object constant

By combining writable:false and configurable:false, you can essentially create a constant (cannot be changed, redefined or deleted) as an object property, like:

```
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

### Prevent extensions

If you want to prevent an object from having new properties added to it, but otherwise leave the rest of the object's properties alone, call Object.preventExtensions(..):

```
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

### Seal

`Object.seal(..)` creates a "sealed" object, which means it takes an existing object and essentially calls `Object.preventExtensions(..)` on it, but also marks all its existing properties as `configurable:false`.

So, not only can you not add any more properties, but you also cannot reconfigure or delete any existing properties (though you can still modify their values)

### Freeze

`Object.freeze(..)` creates a frozen object, which means it takes an existing object and essentially calls `Object.seal(..)` on it, but it also marks all "data accessor" properties as `writable:false`, so that their values cannot be changed.

This give highest level of immutability.


### [[GET]]

the code `myobject.a` performs a [[Get]] operation (kinda like a function call: [[Get]]()) on the myObject. The default built-in [[Get]] operation for an object first inspects the object for a property of the requested name, and if it finds it, it will return the value accordingly. If the property value is not found will return undefined.

### [[PUT]]

When invoking [[Put]], how it behaves differs based on a number of factors, including (most impactfully) whether the property is already present on the object or not.

If the property is present, the [[Put]] algorithm will roughly check:
2. Is the property an accessor descriptor (see "Getters & Setters" section below)?      If so, call the setter, if any.
3. Is the property a data descriptor with writable of false? If so, silently fail in    non-strict mode, or throw TypeError in strict mode.
4. Otherwise, set the value to the existing property as normal.

If the property is not yet present on the object in question, the [[Put]] operation is even more nuanced and complex

### Getters an Setters

When you define a property to have either a getter or a setter or both, its definition becomes an "accessor descriptor" (as opposed to a "data descriptor"). For accessor-descriptors, the value and writable characteristics of the descriptor are moot and ignored, and instead JS considers the set and get characteristics of the property (as well as configurable and enumerable).

```
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// target
	"b",		// property name
	{			// descriptor
		// define a getter for `b`
		get: function(){ return this.a * 2 },

		// make sure `b` shows up as an object property
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

Another example

```
var myObject = {
	// define a getter for `a`
	get a() {
		return this._a_;
	},

	// define a setter for `a`
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

### Existence

a property access like myObject.a may result in an undefined value if either the explicit undefined is stored there or the a property doesn't exist at all. So, if the value is the same in both cases, how else do we distinguish them?

```
var myObject = {
	a: 2
};

("a" in myObject);				// true , 
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

`in` operator checks the object and its prototype
`hasOwnProperty(...)` checks only that object

### Enumerable

Enumerable and non Enumerable can be distinguished using

```
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` non-enumerable
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

`propertyIsEnumerable(..)` tests whether the given property name exists directly on the object and is also enumerable:true.

`Object.keys(..)` returns an array of all enumerable properties, whereas `Object.getOwnPropertyNames(..)` returns an array of all properties, enumerable or not.`

### Iteration

numerically-indexed arrays, iterating over the values is typically done with a standard `for loop`, like:

```
var myArray = [1, 2, 3];

for (var i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```

`for of array`

```
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

`Symbol iterator`

```
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```


