# Scopes

## What is Scope ?

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program state.

**Scope** is well-defined set of rules for storing variables in some location, and for finding those variables at a later time

## Compiler Theory

Javascript is a compiled language. Its is not compiled well is advance or the results of compilation is not portable among various distributed system.the JavaScript engine performs many of the same steps, albeit in more sophisticated ways than we may commonly be aware, of any traditional language-compiler.

Traditional compilation process code will undergo three steps.

`Tokenizing/Lexing` -> `Parsing` -> `Code Generation`

1. **Tokenizing/Lexing:** breaking up a string of characters into meaningful (to the language) chunks, called tokens. For instance, consider the program: `var a = 2;`. This program would likely be broken up into the following tokens: `var`, `a`, `=`, `2`, and `;`.

2) **Parsing:** taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an "AST" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).

3) **Code-Generation:** the process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it's targeting, etc.

## Who does this ?

1. **Engine** : responsible for start-to-finish compilation and execution of our JavaScript program.
2. **Compiler** : one of Engine's friends; handles all the dirty work of parsing and code-generation
3. **Scope** : collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code.

## How does compilation and scoping of variables happen ?

Say we have a code `var a =2` ,

### Compilation

1.Compiler performs **lexing** to break it down into tokens
2.Compiler will **parse** the tokens into a tree.
3.In _code-generation_ phase
a.Compiler asks Scope to see if a variable a already exists for that particular scope collection. If so, Compiler ignores this declaration and moves on. Otherwise, Compiler asks Scope to declare a new variable called a for that scope collection.
4.Compiler then produces code for Engine to later execute, to handle the `a = 2` assignment.

### Execution

While executing the generated code after compilation , The code Engine will first ask Scope if there is a variable called `a` accessible in the current scope collection. If so, Engine uses that variable. If not, Engine looks elsewhere

The type of look-up Engine performs affects the outcome of the look-up.

Two types of look up.
1.Who is the target of assignment (LHS) - ex `var a = 2`
2.Who is the source of assignment (RHS) - ex `console.log(a)`

Example:

```
function foo(a) { // Implicit LHS
    console.log(a) // RHS
}
foo(2) // RHS
```

## Nested Scope

a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, Engine consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

consider

```
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for b cannot be resolved inside the function foo, but it can be resolved in the Scope surrounding it (in this case, the global).

### Rules for traversing nested scope

Engine starts at the currently executing Scope, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

## Errors

### Reference Error

If an RHS look-up fails to ever find a variable, anywhere in the nested Scopes, this results in a `ReferenceError` being thrown by the Engine. It's important to note that the error is of the type `ReferenceError`.

By contrast, if the Engine is performing an LHS look-up and arrives at the top floor (global Scope) without finding it, and if the program is not running in "Strict Mode" then the global Scope will create a new variable of that name in the global scope, and hand it back to Engine.

### Type Error

if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a null or undefined value, then Engine throws a different kind of error, called a TypeError.

## Lexical Scope

lexical scope is scope that is defined at lexing time. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code.

Scope consists of a series of "bubbles" that each act as a container or bucket, in which identifiers (variables, functions) are declared. These bubbles nest neatly inside each other, and this nesting is defined at author-time.

```
function foo(a) { // Bubble A - Global scope

	var b = a * 2; // Bubble B - foo scope

	function bar(c) {
		console.log( a, b, c ); // Bubble C - bar scope
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

Bubble 1 encompasses the global scope, and has just one identifier in it: `foo`.

Bubble 2 encompasses the scope of foo, which includes the three identifiers: `a`, `bar` and `b`.

Bubble 3 encompasses the scope of bar, and it includes just one identifier: `c`.

In the above code snippet, the Engine executes the console.log(..) statement and goes looking for the three referenced variables `a`, `b`, and `c`. It first starts with the innermost scope bubble, the scope of the `bar(..)` function. It won't find a there, so it goes up one level, out to the next nearest scope bubble, the scope of `foo(..)`. It finds a there, and so it uses that a. Same thing for b. But c, it does find inside of `bar(..)`.

No matter where a function is invoked from, or even how it is invoked, its lexical scope is only defined by where the function was declared.

### Shadowing

The same identifier name can be specified at multiple layers of nested scope, which is called "shadowing" (the inner identifier "shadows" the outer identifier). Regardless of shadowing, scope look-up always starts at the innermost scope being executed at the time, and works its way outward/upward until the first match, and stops.

### Global variables

Global variables are also automatically properties of the global object (window in browsers, etc.), so it is possible to reference a global variable not directly by its lexical name, but instead indirectly as a property reference of the global object. ex `window.a`


## Scope From Functions

Javascript has function based scope. Function scope encourages the idea that all variables belong to the function, and can be used and reused throughout the entirety of the function (and indeed, accessible even to nested scopes).


### Hiding in plain Scope

If a code is wrapped by a function, The practical result is to create a scope bubble around the code in question, which means that any declarations (variable or function) in that code will now be tied to the scope of the new wrapping function, rather than the previously enclosing scope. In other words, you can "hide" variables and functions by enclosing them in the scope of a function.

### Advantages of Hiding

1. Promote "Principle of Least Privilege" (principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and "hide" everything else.)

2. to avoid unintended collision between two different identifiers with the same name but different intended usages. 

### Global namespace to avoid collision 

 Javascript libraries typically will create a single variable declaration, often an object, with a sufficiently unique name, in the global scope. This object is then used as a "namespace" for that library, where all specific exposures of functionality are made as properties of that object (namespace), rather than as top-level lexically scoped identifiers themselves.

```
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

## Function as scopes

Wrapping a function over a piece of code hides the code and is useful but not a very ideal approach.

Below example

```
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```
1. Need to create a foo function which pollutes global scope
2. Need to explicitly call foo function

What if a function doesnt have a name and we need not call the function explicitly ?

Javascript offers the below solution - IIFE (Immediately Invoked Function Expression)

```
ar a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2

```

IIFE doesnt need a name but using anonymous function as below drawbacks.

1.Anonymous functions have no useful name to display in stack traces, which can make debugging more difficult.

2.Without a name, if the function needs to refer to itself, for recursion, etc., the deprecated arguments.callee reference is unfortunately required. Another example of needing to self-reference is when an event handler function wants to unbind itself after it fires.

3.Anonymous functions omit a name that is often helpful in providing more readable/understandable code. A descriptive name helps self-document the code in question.

### IIFE variant 1

```
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

###  IIFE variant 2

```
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window ); // Arguments passed to IIFE

console.log( a ); // 2
```

### IIFE variant 3

```
(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

## Block Scope

`var` is always function scoped inside the function where it is declared. Declaring a `var` inside a for loop/ if block is block scoped to entire function. 

Always declare `var` closer to where its being used.

`const` and `let` are block `{}` scoped.

```
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

variables declared inside try and catch block are scoped inside the block.

```
try {
	undefined(); // illegal operation to force an exception!
}
catch (err) {
	console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

## Hoisting

All declarations, both variables and functions, are processed first, before any part of your code is executed.

```
a = 2;

var a;

console.log( a ); // 2
```

When you see var a = 2;, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: var a; and a = 2;. The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left in place for the execution phase.

Our first snippet then should be thought of as being handled like this:

```
var a;
a = 2;

console.log( a ); // 2
```

```
console.log( a );

var a = 2;
```

should be thought of as being handled like below:

```
var a;
console.log( a );

a = 2;
```

one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are "moved" from where they appear in the flow of the code to the top of the code. This gives rise to the name "Hoisting".

Function declarations are hoisted

```
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

Function expressions are not hoisted

```
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
	// ...
};
```

Function declarations are hoisted first followed by variables declarations.

```
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

`1` is printed instead of `2`! This snippet is interpreted by the Engine as:

```
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```


Notice that `var foo` was the duplicate (and thus ignored) declaration, even though it came before the function `foo()...` declaration, because function declarations are hoisted before normal variables.

While multiple/duplicate var declarations are effectively ignored, subsequent function declarations do override previous ones.
