
# Tag funcions

Consider

```
var name = "kyle";
var orderNumber = "123";
var total = 397.1;

var msg = `Hello , ${name}, your \
order (#${orderNumber}) was $${total}.`;
```

We can put an expression in front of the string literal and it can act as a function call.

```
var name = "kyle";
var orderNumber = "123";
var total = 397.1;

var msg = foo `Hello , ${name}, your \ //-----------------> foo expression
order (#${orderNumber}) was $${total}.`;
```

where foo is a function
gets the string literals as arguments (array)
and interpolated values as values (collected using ... operator)

see below

```
function foo(strings, ...values){
    var str = "";
    for (var i=0; i< strings.length; i++ ){
        if(i > 0){
            if(typeof values[i-1] == "number"){
                str += values[i-1].toFixed(2);
            }else {
                str +=  values[i-1];
            }
        }
        str += strings[i];
    }
    return str;
}

var name = "kyle";
var orderNumber = "123";
var total = 397.1;

var msg = foo `Hello , ${name}, your \ //-----------------> foo expression
order (#${orderNumber}) was $${total}.`;
```

The `foo` function is used as preprocessor for the final string

In above example 

string[] = ['Hello , ', 'your \order (#', ') was $','.']

...values = [name,orderNumber,total];

**Note**: Always the string value will have > 1 string than the number of interpolation values
