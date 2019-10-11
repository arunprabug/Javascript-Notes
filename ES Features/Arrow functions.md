# Arrow functions

```
function foo(x,y) {
	return x + y;
}

// versus

var foo = (x,y) => x + y;
```

More variations

```
var f1 = () => 12;
var f2 = x => x * 2;
var f3 = (x,y) => {
	var z = x * 2 + y;
	y++;
	x *= 3;
	return (x + y + z) / 2;
};
```

Arrow functions are always function expressions; there is no arrow function declaration. It also should be clear that they are anonymous function expressions.

Note: All the capabilities of normal function parameters are available to arrow functions, including default values, destructuring, rest parameters, and so on.

## Not Just Shorter Syntax, But this

Most of the popular attention toward => has been on saving those precious keystrokes by dropping function, return, and { .. } from your code.

But there's a big detail we've skipped over so far. I said at the beginning of the section that => functions are closely related to this binding behavior. In fact, => arrow functions are primarily designed to alter this behavior in a specific way, solving a particular and common pain point with this-aware coding.

```
var controller = {
	makeRequest: function(..){
		var self = this;

		btn.addEventListener( "click", function(){
			// ..
			self.makeRequest(..);
		}, false );
	}
};
```

We used the var self = this hack, and then referenced self.makeRequest(..), because inside the callback function we're passing to addEventListener(..), the this binding will not be the same as it is in makeRequest(..) itself. In other words, because this bindings are dynamic, we fall back to the predictability of lexical scope via the self variable.

Herein we finally can see the primary design characteristic of => arrow functions. Inside arrow functions, the this binding is not dynamic, but is instead lexical. In the previous snippet, if we used an arrow function for the callback, this will be predictably what we wanted it to be.

```
var controller = {
	makeRequest: function(..){
		btn.addEventListener( "click", () => {
			// ..
			this.makeRequest(..);
		}, false );
	}
};
```

Lexical this in the arrow function callback in the previous snippet now points to the same value as in the enclosing makeRequest(..) function. In other words, => is a syntactic stand-in for var self = this.

In cases where var self = this (or, alternatively, a function .bind(this) call) would normally be helpful, => arrow functions are a nicer alternative operating on the same principle. 

