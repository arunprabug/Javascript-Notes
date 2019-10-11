# Generators

All functions in Javascript has a run to completion semantics. What does it mean is if a function starts running , it will always run to the ends and finish before any other function has an opportunity to come in and start running. At any given moment only one function is active and executing.Nobody can come in preemptively interrupt this function.This is call **Run to completion** semantics.

Generators are functions but doesnt have run to finish semantics. We are trying to create  a syntatic form of declaring a state machine.State machines are ways of having pattern series of flow from one state to another and declaratively listening to all the state and their transitions.

We can say Generator is a pausable function. It runs with a Yield keyword and when the yield key shows up even in the middle of the expression everything freezes. It will wait in the paused state indefinitely until someother actor comes along and says its time to resume.

We can thinks of Generator as a function that can pause and resume as many times as necessary.While the generator is paused, on the inside of generator nothing is happening and its localized blocking (only inside the generator function) and it doesnt not affect the overall program.

```
function* gen(){
    console.log("Hello");
    yield; 
    console.log("world");
}

var it = gen();
it.next(); //Hello
it.next(); //World
```
The * symbol identifies the function as a generator function.
`yield` is the pause button which blocks the code, and its cooperative blocking and not preemptive blocking.

 Executing the `gen()` function does not run  the generator instead it produces an iterator object. The purpose of iterator is to step through the control of generator from the outside.We cannot pause the function generator from the outside using iterator but we can step the control until the next `yield`

1. first `it.next()` code will start running the generator, prints "Hello" and executes until the first `yield` where the generator is blocked and control is resumed back to first `it.next()`.
3. Now control proceeds to next `it.next()` again the generator is resumed and "world" is printed.  

The call to second `it.next()` can happen immediately or after sometime so we can synchronously step through the generator.

## Messaging

**Messaging passing with yield**

Consider

```
function* main(){
    yield 1;
    yield 2;
    yield 3;
}

var it = main();

it.next();  // {value:1 , done: false}
it.next();  // {value:2 , done: false}
it.next();  // {value:3 , done: false}

it.next();  // {value:undefined , done: true}
```
above case `yield` yields a value .

1. `yield 1` returns the control to first `it.next()` and also returns a object of format `{value:1 , done: false}`.where value is the `value` yielded and `done` which says if the generator is complete

2. If `it.next()` is called and the generator is complete then we get return output of the function as `value` and `done` as `false` . If the function doesnt return any output then value will be `undefined`. When `return` is encountered anywhere in the function generator will be complete instantly.

3. We can say `yield` gives `{value:some value yielded, done:false}`  and `return` gives `{value:some value yielded, done:true}`

## 2 Way Message passing with Yield (Coroutines)

```
function coroutine(g) {
    var it = g();
    return function() {
        return it.next().apply(it,arguments)
    }
}
```

```
var run = coroutine(function* (){
    var x = 1 + (yield); // -----------------> braces mandatory
    var y = 1 + (yield);
    yield(x + y);
});

run();
run(10);
console.log(
    "Meaning of life " + run(30).value
);
```

`var x = 1 + (yield)` in above example, the `yield` keyworkd is like a placeholder and it asks for a value from outside world.

1. `run()` will execute `it.next()` on the above generator function
2. control goes to `var x = 1 + (yield)` , a yield expression is encountered
3. `yield` pause the execution and asks the outside world for a value to complete the expression
4. control comes back to `run()`
5. `run(10)` (it.next())is executed and control goes to `var x = 1 + (yield)`
6.  Value `10` is the missing value that `yield` asked for, the expression is evaluated and next line `var y = 1 + (yield)` is executed
7. `yield` pause the execution and asks the outside world for a value to complete the expression
8. control comes back to `run(10)`
9. control comes back to `console.log` and `run(30).value` (it.next) is executed
10. Value `10` is the missing value that `yield` asked for, the expression is evaluated and next line yield(x+y) is executed
11. {value:42, done:false} is sent as yielded value 
12. Control comes back to `console.log` and it prints the value.

Above code generator is not finished, sometimes we design generators to never finish.

Generators are those which produces some value thats why they are called Generators.

## Async Generators

Consider below example

```
function foo(x,y,cb) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		cb
	);
}

foo( 11, 31, function(err,text) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( text );
	}
} );
```

If we wanted to express this same task flow control with a generator, we could do:

```
function foo(x,y) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
				// throw an error into `*main()`
				it.throw( err );
			}
			else {
				// resume `*main()` with received `data`
				it.next( data );
			}
		}
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

var it = main();

// start it all up!
it.next();
```

At first glance, this snippet is longer, and perhaps a little more complex looking, than the callback snippet before it. But don't let that impression get you off track. The generator snippet is actually much better! But there's a lot going on for us to explain.

First, let's look at this part of the code, which is the most important:

```
var text = yield foo( 11, 31 );
console.log( text );

```

Think about how that code works for a moment. We're calling a normal function `foo(..)` and we're apparently able to get back the text from the Ajax call, even though it's asynchronous.

It's because of the yield used in a generator.

In yield `foo(11,31)`, first the `foo(11,31)` call is made, which returns nothing (aka undefined), so we're making a call to request data, but we're actually then doing `yield undefined`. That's OK, because the code is not currently relying on a yielded value to do anything interesting.

So, the generator pauses at the yield, essentially asking the question, "what value should I return to assign to the variable text?" Who's going to answer that question?

Look at `foo(..)`. If the Ajax request is successful, we call:

```
it.next( data )
```

That's resuming the generator with the response data, which means that our paused `yield` expression receives that value directly, and then as it restarts the generator code, that value gets assigned to the local variable text.

Take a step back and consider the implications. We have totally synchronous-looking code inside the generator (other than the yield keyword itself), but hidden behind the scenes, inside of `foo(..)`, the operations can complete asynchronously.

That's huge! That's a nearly perfect solution to our previously stated problem with callbacks not being able to express asynchrony in a sequential, synchronous fashion that our brains can relate to.

In essence, we are abstracting the asynchrony away as an implementation detail, so that we can reason synchronously/sequentially about our flow control: "Make an Ajax request, and when it finishes print out the response." And of course, we just expressed two steps in the flow control, but this same capability extends without bounds, to let us express however many steps we need to.

## Synchronous Error Handling

```
try {
	var text = yield foo( 11, 31 );
	console.log( text );
}
catch (err) {
	console.error( err );
}
```

How does this work? The `foo(..)` call is asynchronously completing, and doesn't try..catch fail to catch asynchronous errors

We already saw how the yield lets the assignment statement pause to wait for `foo(..)` to finish, so that the completed response can be assigned to text. The awesome part is that this yield pausing also allows the generator to catch an error.

We have solved **non local no sequential reasonability issues**

## Generator + Promises

Instead of yielding a `undefined` we are going to yield a `promises`

When the promise resolves we resume the generator.

it.next () -> yield promise -> in promise then(when promise resolves) -> it.next()

See below

```
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

foo( 11, 31 )
.then(
	function(text){
		console.log( text );
	},
	function(err){
		console.error( err );
	}
)
```

In our earlier generator code for the running Ajax example, `foo(..)` returned nothing (undefined), and our iterator control code didn't care about that yielded value.

But here the Promise-aware `foo(..)` returns a promise after making the Ajax call. That suggests that we could construct a promise with `foo(..)` and then yield it from the generator, and then the iterator control code would receive that promise.

But what should the iterator do with the promise?

It should listen for the promise to resolve (fulfillment or rejection), and then either resume the generator with the fulfillment message or throw an error into the generator with the rejection reason.

Let me repeat that, because it's so important. The natural way to get the most out of Promises and generators is to yield a Promise, and wire that Promise to control the generator's iterator.

Let's give it a try! First, we'll put the Promise-aware `foo(..)` together with the generator `*main()`

```
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
```

The most powerful revelation in this refactor is that the code inside `*main()` did not have to change at all! Inside the generator, whatever values are yielded out is just an opaque implementation detail, so we're not even aware it's happening, nor do we need to worry about it.

But how are we going to run `*main()` now? We still have some of the implementation plumbing work to do, to receive and wire up the yielded promise so that it resumes the generator upon resolution.

```
var it = main();

var p = it.next().value;

// wait for the `p` promise to resolve
p.then(
	function(text){
		it.next( text );
	},
	function(err){
		it.throw( err );
	}
);
```

Actually, that wasn't so painful at all, was it?

This snippet should look very similar to what we did earlier with the manually wired generator controlled by the error-first callback. Instead of an if (err) { it.throw.., the promise already splits fulfillment (success) and rejection (failure) for us, but otherwise the iterator control is identical.

Most importantly, we took advantage of the fact that we knew that `*main()` only had one Promise-aware step in it. What if we wanted to be able to Promise-drive a generator no matter how many steps it has? We certainly don't want to manually write out the Promise chain differently for each generator! What would be much nicer is if there was a way to repeat (aka "loop" over) the iteration control, and each time a Promise comes out, wait on its resolution before continuing.

Also, what if the generator throws out an error (intentionally or accidentally) during the `it.next(..)` call? Should we quit, or should we catch it and send it right back in? Similarly, what if we `it.throw(..)` a Promise rejection into the generator, but it's not handled, and comes right back out?

## Promise Aware Generator runner

```
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	// initialize the generator in the current context
	it = gen.apply( this, args );

	// return a promise for the generator completing
	return Promise.resolve()
		.then( function handleNext(value){
			// run to the next yielded value
			var next = it.next( value );

			return (function handleResult(next){
				// generator has completed running?
				if (next.done) {
					return next.value;
				}
				// otherwise keep going
				else {
					return Promise.resolve( next.value )
						.then(
							// resume the async loop on
							// success, sending the resolved
							// value back into the generator
							handleNext,

							// if `value` is a rejected
							// promise, propagate error back
							// into the generator for its own
							// error handling
							function handleErr(err) {
								return Promise.resolve(
									it.throw( err )
								)
								.then( handleResult );
							}
						);
				}
			})(next);
		} );
}
```
How would you use `run(..)` with `*main()` in our running Ajax example?

```
function *main() {
	// ..
}

run( main );
```



## ES7 Async and Await

The preceding pattern -- generators yielding Promises that then control the generator's iterator to advance it to completion -- is such a powerful and useful approach, it would be nicer if we could do it without the clutter of the library utility helper (aka run(..)).

```
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}

async function main() {
	try {
		var text = await foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}

main();
```

As you can see, there's no `run(..)` call (meaning no need for a library utility!) to invoke and drive `main()` -- it's just called as a normal function. Also, `main()` isn't declared as a generator function anymore; it's a new kind of function: async function. And finally, instead of yielding a Promise, we await for it to resolve.

The async function automatically knows what to do if you await a Promise -- it will pause the function (just like with generators) until the Promise resolves. We didn't illustrate it in this snippet, but calling an async function like `main()` automatically returns a promise that's resolved whenever the function finishes completely.










