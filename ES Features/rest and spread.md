# Spread/ Rest

ES6 introduces a new ... operator that's typically referred to as the spread or rest operator, depending on where/how it's used. Let's take a look:

```
function foo(x,y,z) {
	console.log( x, y, z );
}

foo( ...[1,2,3] );				// 1 2 3
```

When ... is used in front of an array (actually, any iterable), it acts to "spread" it out into its individual values.

You'll typically see that usage as is shown in that previous snippet, when spreading out an array as a set of arguments to a function call. In this usage, ... acts to give us a simpler syntactic replacement for the apply(..) method, which we would typically have used pre-ES6 as:

```
foo.apply( null, [1,2,3] );		// 1 2 3

```

But `...` can be used to spread out/expand a value in other contexts as well, such as inside another array declaration:

```
var a = [2,3,4];
var b = [ 1, ...a, 5 ];

console.log( b );					// [1,2,3,4,5]
```

The other common usage of `...` can be seen as essentially the opposite; instead of spreading a value out, the `...` gathers a set of values together into an array. Consider:

```
function foo(x, y, ...z) {
	console.log( x, y, z );
}

foo( 1, 2, 3, 4, 5 );			// 1 2 [3,4,5]
```

if you don't have any named parameters, the ... gathers all arguments:

```
function foo(...args) {
	console.log( args );
}

foo( 1, 2, 3, 4, 5);			// [1,2,3,4,5]
```

