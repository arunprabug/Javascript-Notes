# Thunks

## Synchronous Thunk:

A Thunk is a function that has everything already that it needs to do to give a value back. Its a function with some closured state  keeping track of some values and gives those things back when you call that function..

```
function add (x,y) {
    return x+y;
}

var thunk = function (){
    return add(10,15);
};

thunk(); //25
thunk(); //25
```

Thunk is a container around that particular collection of state(a function), we can pass this container around in our program and invoke it and it will give the value back.

## Asynchronous thunk.

Its a function that doesnt need any arguments to do its job except we need to pass a callback to get its value back.

```
function addAsync (x,y, cb) {
    setTimeout(function(){
        cb(x+y)
    },1000)
}

var thunk = function (cb){
    addAsync(10,15,cb);
};

thunk(function sum(){
    sum;    //25
});
```

The above pattern is very powerful because from the outside world we need not worry or care about the value will be available now or not (will it be available after sometime).

We have normalised time out of the equation, the wrapper around the value is time independent.(Time is the most complex factor of state in our program)

## Promise - Thunk
A promise is time independent wrapper around a value and the concept behind is thunks. Thunks are promises without special API

## Nested Thunks

```
var get10 = makeThunk(getData,10);
var get30 = makeThunk(getData,30);


get10(function(num1){
    var x = 1 + num1;
    get30(function(num2){
        var y = 1 + num2;

        var getAnswer = makeThunk(getData, 
            "Meaning of life: " + (x+y)
        );

        getAnswer(function(answer){
            console.log(answer);
        })
    })
})
```
**Lazy Thunk** - A thunk which doesnt do the work until you call it first time.
**Active Thunk** - A thunk which does it work and held it response until you pass the callback.

A thunk uses closure to maintain state by eliminating time.











