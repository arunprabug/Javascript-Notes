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
