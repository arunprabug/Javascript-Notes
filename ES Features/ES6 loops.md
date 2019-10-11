# for..of loop

Joining the for and for..in loops from the JavaScript we're all familiar with, ES6 adds a for..of loop, which loops over the set of values produced by an iterator.

The value you loop over with for..of must be an iterable, or it must be a value which can be coerced/boxed to an object that is an iterable. An iterable is simply an object that is able to produce an iterator, which the loop then uses.

```
var a = ["a","b","c","d","e"];

for (var idx in a) {
	console.log( idx );
}
// 0 1 2 3 4

for (var val of a) {
	console.log( val );
}
// "a" "b" "c" "d" "e"
```

Standard built-in values in JavaScript that are by default iterables (or provide them) include:

Arrays
Strings
Generators
Collections / TypedArrays 



