# ES.Next

## Block level declarations

### Let declarations

```
var a = 2;

{
	let a = 3;
	console.log( a );	// 3
}

console.log( a );		// 2
```

It's not very common or idiomatic thus far in JS to use a standalone { .. } block, but it's always been valid. And developers from other languages that have block scoping will readily recognize that pattern.

Accessing a let-declared variable earlier than its let .. declaration/initialization causes an error, whereas with var declarations the ordering doesn't matter (except stylistically).

```
{
	console.log( a );	// undefined
	console.log( b );	// ReferenceError!

	var a;
	let b;
}
```

**let + for**

```
var funcs = [];

for (let i = 0; i < 5; i++) {
	funcs.push( function(){
		console.log( i );
	} );
}

funcs[3]();		// 3
```

## Const Declarations

What exactly is a constant? It's a variable that's read-only after its initial value is set. Consider:

```
{
	const a = 2;
	console.log( a );	// 2

	a = 3;				// TypeError!
}
```

You are not allowed to change the value the variable holds once it's been set, at declaration time. A `const` declaration must have an explicit initialization. If you wanted a constant with the undefined value, you'd have to declare `const a = undefined` to get it.

`Constants` are not a restriction on the value itself, but on the variable's assignment of that value. In other words, the value is not frozen or immutable because of `const`, just the assignment of it. If the value is complex, such as an `object or array`, the contents of the value can still be modified:

```
{
	const a = [1,2,3];
	a.push( 4 );
	console.log( a );		// [1,2,3,4]

	a = 42;					// TypeError!
}
```

The a variable doesn't actually hold a constant array; rather, it holds a constant reference to the array. The array itself is freely mutable.

`const` can be used with variable declarations of `for, for..in, and for..of` loops (see "for..of Loops"). However, an error will be thrown if there's any attempt to reassign, such as the typical i++ clause of a for loop.





