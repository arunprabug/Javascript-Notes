# Object literal extensions

## Concise Properties

```
var x = 2, y = 3,
	o = {
		x: x,
		y: y
	};
```

If it's always felt redundant to say x: x all over, there's good news. If you need to define a property that is the same name as a lexical identifier, you can shorten it from x: x to x. Consider:

```
var x = 2, y = 3,
	o = {
		x,
		y
	};
```

## Concise methods

In a similar spirit to concise properties we just examined, functions attached to properties in object literals also have a concise form, for convenience.

```
var o = {
	x: function(){
		// ..
	},
	y: function(){
		// ..
	}
}
```

Below ES6 way

```
var o = {
	x() {
		// ..
	},
	y() {
		// ..
	}
}
```

concise methods imply anonymous function expressions.

## ES5 Getter and Setter

ES5 defined getter/setter literals forms, but they didn't seem to get used much, mostly due to the lack of transpilers to handle that new syntax (the only major new syntax added in ES5, really).

```
var o = {
	__id: 10,
	get id() { return this.__id++; },
	set id(v) { this.__id = v; }
}

o.id;			// 10
o.id;			// 11
o.id = 20;
o.id;			// 20

// and:
o.__id;			// 21
o.__id;			// 21 -- still!
```

## Computed property names

You've probably been in a situation like the following snippet, where you have one or more property names that come from some sort of expression and thus can't be put into the object literal:

```
var prefix = "user_";

var o = {
	baz: function(..){ .. }
};

o[ prefix + "foo" ] = function(..){ .. };
o[ prefix + "bar" ] = function(..){ .. };
```

ES6 adds a syntax to the object literal definition which allows you to specify an expression that should be computed, whose result is the property name assigned. Consider:

```
var prefix = "user_";

var o = {
	baz: function(..){ .. },
	[ prefix + "foo" ]: function(..){ .. },
	[ prefix + "bar" ]: function(..){ .. }
	..
};
```





