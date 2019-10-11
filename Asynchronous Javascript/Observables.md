# Observables

## Event promises

Lets say we want to represent click of a button using promises.

```
var p1 = new Promise(function (resolve,reject){
    $('#btn').click(function(evt){
        var className = evt.target.className;
        if(/fooBar/.test(className)){
            resolve(className);
        }else{
            reject();
        }
    });
})

p1.then(function(className){
    console.log(className);
})
```
The problem with the approach is **Promises are resolved only once**

Only first time the event will work , after that click on the button wont have any effect.

Promises by themselves are not going to model well in event oriented world.

### Solution

```
$('#btn').click(function(evt){

    [evt] // put my events into an list, synchronous mindset
        .map(function(evt){
            evt.target.className;
        })
        .filter(function(className){
            return /fooBar/.test(className)
        })
        .forEach(function(className){
            console.log(className);
        })
    });

```

The response to the event needs to be conflated with the setup for the event , we have producer , consumer problem in same spot. We need to bridge the gap so we can separate the concerns.

## Solution: Observables

Library - RXJS by microsoft.

Obserables is like a chain of calculated fields in a Spreadsheet.A input in one field might calculate value in some other field. 

We are subscribing and responding to data.

This is the genesis of reactive programming, Basically the source is an event source.  Once i get the event its going to go in series of steps, some steps are going to be direct like responsive and calculate somthing. But some of them might be synchronous steps. There will be some propagation delay.

Reactive programming is a way to model a piece of data coming in one side (from producer or event emitter) and all the steps it takes to propogate through the system.  Its a data flow mechanism.
It is built from ground up to respond to events.

An observable is an adapter hooked onto a event source that produces a promise everytime a new event comes through. But it does so in separate way so that i can setup my event source in an entirely different part of my location , i can declaratively say what is my data flow.

Example

```
var obsv = RX.Observable.fromEvent(btn,"click");

obsv.
    map(function mapper(evt){
        return evt.target.className;
    })
    .filter(function filterer(className){
        return /fooBar/.test(className)
    })
    .distinctUntilChanged()
    .subscribe(function(data){
        var className = data[i];
        console.log(className);
    });
```











