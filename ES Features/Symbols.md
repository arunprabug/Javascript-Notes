# Symbol

Symbol is  unique globally unguessable value within the context of programming.

You are not supposed to think what is under the covers. As a programmer you just opaquely use the symbol, you dont know what value it is, you cant find out what value it is and you are not supposed to care what value it is.

You are supposed to use it as a symbolic place holder for  unique globally unguessable value.

Heres how to create a symbol

```
var sym = Symbol( "some optional description" );

typeof sym;		// "symbol"
```

Some things to note:

1. You cannot and should not use new with Symbol(..). It's not a constructor, nor are you producing an object.
2. The parameter passed to Symbol(..) is optional. If passed, it should be a string that gives a friendly description for the symbol's purpose.
3. The typeof output is a new value ("symbol") that is the primary way to identify a symbol.

The description, if provided, is solely used for the stringification representation of the symbol:

```
sym.toString();		// "Symbol(some optional description)"
```


The internal value of a symbol itself -- referred to as its name -- is hidden from the code and cannot be obtained. You can think of this symbol value as an automatically generated, unique (within your application) string value.

But if the value is hidden and unobtainable, what's the point of having a symbol at all?

The main point of a symbol is to create a string-like value that can't collide with any other value. So, for example, consider using a symbol as a constant representing an event name:

```
const EVT_LOGIN = Symbol( "event.login" );

```

You'd then use `EVT_LOGIN` in place of a generic string literal like "event.login":

```
evthub.listen( EVT_LOGIN, function(data){
	// ..
} );
```

The benefit here is that `EVT_LOGIN` holds a value that cannot be duplicated (accidentally or otherwise) by any other value, so it is impossible for there to be any confusion of which event is being dispatched or handled.

### Symbol as normal object properties

If a symbol is used as a property/key of an object, it's stored in a special way so that the property will not show up in a normal enumeration of the object's properties:

```
var o = {
	foo: 42,
	[ Symbol( "bar" ) ]: "hello world",
	baz: true
};

Object.getOwnPropertyNames( o );	// [ "foo","baz" ]
```

To retrieve an object's symbol properties:

```
Object.getOwnPropertySymbols( o );	// [ Symbol(bar) ]
```

This makes it clear that a property symbol is not actually hidden or inaccessible, as you can always see it in the `Object.getOwnPropertySymbols(..)` list.

## Built in Symbols

ES6 comes with a number of predefined built-in symbols that expose various meta behaviors on JavaScript object values. However, these symbols are not registered in the global symbol registry, as one might expect.

Instead, they're stored as properties on the `Symbol` function object. For example, in the "for..of" section earlier in this chapter, we introduced the `Symbol.iterator` value:

```
var a = [1,2,3];

a[Symbol.iterator];			// native function
```

The specification uses the `@@` prefix notation to refer to the built-in symbols, the most common ones being: `@@iterator`, `@@toStringTag`, `@@toPrimitive`. Several others are defined as well, though they probably won't be used as often.











