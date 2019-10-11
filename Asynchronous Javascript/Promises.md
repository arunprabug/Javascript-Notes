# Promises

The callback issue we want to address first is the inversion of control, the trust that is so fragilely held and so easily lost.

We wrap up the continuation of our program in a callback function, and hand that callback over to another party (potentially even external code) and just cross our fingers that it will do the right thing with the invocation of the callback.

We do this because we want to say, "here's what happens later, after the current step finishes."

But what if we could uninvert that inversion of control? What if instead of handing the continuation of our program to another party, we could expect it to return us a capability to know when its task finishes, and then our code could decide what to do next?

This paradigm is called Promises.

## What is promise ?

Promise is the codification of the idea that we need a placeholder to eliminate time as the concern wrapped around a value , we need a container around it. In programming they call it **Future value**. Its the value which eventually in a some unspecified time will become fulfilled. But in the mean time we still need to reason about it exatly the same way.

The concept came from "E" programming language.

Promises are basically a *Monad* in functional programming paradigm.

Promsies uninvert the paradigm we tradionally used to interact with some other mechanism which is Asynchronous in nature.

Instead of calling a callback provided by the third party utility we ask the third party to give us a event listner to which we can subscribe/listen to the **completion event/error event**.We ask the third party to fire the event when its ready and we will be notified for us to act and move on.

```
function finish(){
    chargeCreditCard(purchaseInfo)
    showThankYouPage();
}

function error(err){
    logStatsError(err);
    finish();
}

 var listener = trackCheckout(purchaseInfo)

 listener.on("completion",finish)
 listener.on("error",error)
```

The inversion from normal callback-oriented code should be obvious, and it's intentional. Instead of passing the callbacks to `trackCheckout(..)`, it returns an event capability we call evt, which receives the callbacks.

Callbacks themselves represent an inversion of control. So inverting the callback pattern is actually an inversion of inversion, or an uninversion of control -- restoring control back to the calling code where we wanted it to be in the first place.

**Note**: The Promise resolution "events" we listen for aren't strictly events (though they certainly behave like events for these purposes)

From above example In Actual promise API    we dont call the event as **completion** event instead we call it as **then** event

```
function trackCheckout(info) {
    return new Promise(
        function (resolve, reject) {
            // attempt to call checkout

            //if successful, call resolve
            //otherwise , call reject
        }
    )
}
```

promise can have only two outcomes, resolve or reject.

**Registering to thenable event**

```
function finish(){
    chargeCreditCard(purchaseInfo)
    showThankYouPage();
}

function error(err){
    logStatsError(err);
    finish();
}

 var listener = trackCheckout(purchaseInfo)

 promise.then(
     finish,
     error
);

```

## Still Callbacks?

Promise are designed to instill Trust

1. Only resolved once
2. either success Or error
3. messages passed/kept
4. exceptions become errors
5. immutable once resolved

Promises are pattern to manage callback in trustful fashion.

**immutable once resolved** - Means i can send that container anywhere in my system or third party and i need not worry about it.

 ## Flow control

 ### Sequentail flow control

 ```
 doFirstThing
   then doSecondThing
   then doThirdThing
   then complete
 or error   
 ```

 Promises allow us to manage sequential flow control is **chaining promises**

 Return promise from sucess handler of previous promise.

 ```
 doFirstThing()
  .then(function(){
      return doSecondThing();
  }) 
   .then(function(){
       return doThirdThing();
   }) 
   .then(complete, error)
 ```
 
 Promises are not nested like callback chain

 ## then(..) and catch(..)

Each Promise instance (not the Promise API namespace) has `then(..)` and `catch(..)` methods, which allow registering of fulfillment and rejection handlers for the Promise. Once the Promise is resolved, one or the other of these handlers will be called, but not both, and it will always be called asynchronously

`then(..)` takes one or two parameters, the first for the fulfillment callback, and the second for the rejection callback. If either is omitted or is otherwise passed as a non-function value, a default callback is substituted respectively. The default fulfillment callback simply passes the message along, while the default rejection callback simply rethrows (propagates) the error reason it receives.

`catch(..)` takes only the rejection callback as a parameter, and automatically substitutes the default fulfillment callback, as just discussed. In other words, it's equivalent to `then(null,..)`

```
p.then( fulfilled );

p.then( fulfilled, rejected );

p.catch( rejected ); // or `p.then( null, rejected
```

`then(..)` and `catch(..)` also create and return a new promise, which can be used to express Promise chain flow control. If the fulfillment or rejection callbacks have an exception thrown, the returned promise is rejected. If either callback returns an immediate, non-Promise, non-thenable value, that value is set as the fulfillment for the returned promise. If the fulfillment handler specifically returns a promise or thenable value, that value is unwrapped and becomes the resolution of the returned promise.

 ## Promise Abstractions

 ### Promise.all (promise gate)

 ```
 Promise.all([
     doTask1a(),
     doTask1b(),
     doTask1c()
 ])
 .then(function(results){
     return doTask2(
         Math.max(
             results[0],
             results[1],
             results[2]
         );
     );
 });
 ```

 ### Promise.race (promise timeout)

 ```
 var p = trySomeAsyncThing();
 Promise.race([
     p,
     new Promise(function(_,reject){
         setTimeout(function(){
            reject("Timeout!!"); 
         },3000)
     })
 ])
 .then(
     success,
     error
 )
 ```












