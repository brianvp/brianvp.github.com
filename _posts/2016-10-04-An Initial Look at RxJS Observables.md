---
title: 'An Initial Look at RxJS Observables'
date: 2016-10-04T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/10/04/An-Initial-Look-at-RxJS-Observables/
categories:
  - Research
tags:
  - Angular2
  - RxJS
  - Observables
  - ReactiveX
---

A major change in Angular2 from Angular1 is a transition from using `promises` to `observables` in the implementation of the new `Http` angular module.  You can listen to angular team member Rob Wormald discuss this change in an [April 2016 talk](https://youtu.be/JLsR75sf9OY).  

Observables are part of [ReactiveX](http://reactivex.io/).  This is a major cross-platform set of libraries designed to work with several languages. The version Angular uses is called `RxJS`, also known as [Reactive Extensions for JavasScript](https://github.com/Reactive-Extensions/RxJS). 

### Angular 1 $http.get() using Promises

```javascript
$http({
  method: 'GET',
  url: 'http://example.com/api/getStuff'
}).then(function successCallback(response) {
   //do stuff
  });
```

### Angular 2 http.get() using Observables

```javascript
Observable<string[]> observable =  this.http.get('http://example.com/api/getStuff')
                  .map(res: Response => res.json() as string[]);

var observer = observable.subscribe(function(data){
    //do stuff
});
```

Both Promises and Observables are mechanisms for responding to asynchronous events. Promises are designed to retrieve data exactly once.  Once the `.then()` fires, we're done.  Observables are different.  They have the ability to send data multiple times.  Notice in the above example we "subscribe" to the observable.  This is the equivalent of the `.then()` handler in a promise. However, the observable can call this handler *multiple* times! 

Here's an example of how this works:

First, we create an `Observable`

```javascript
var source = Rx.Observable.create(function(observer){
    var itemsToEmit = 5;
    var itemsEmitted = 0;
    
    while (itemsEmitted < itemsToEmit){
        observer.onNext(Math.floor(Math.random() * (101-1) + 1));
        itemsEmitted += 1;
    }
    
    observer.onCompleted();
})
```

The observable is where the async operations happen.  At some point, the Observable needs to pass along some information to the `observer`.  This is done with the `.onNext()` function.  Notice that this is called multiple times. When this happens we say that we are **emitting** data to the observer.  It's the observer's job to decide what to do with this data, which is handled in the subscription:

```Javascript
   source.subscribe(
    function(emittedValue){
        document.getElementById('subscriptionOneOutput').innerHTML += emittedValue + ', ';
    });
```

Each time `.onNext()` is called, the `function(emittedValue)` handler is called.  The observer defines three event handlers for `.onNext()`, `onError()`, and `onCompleted()`.  We can pass these inline to a `.subscribe()`, or create the observer directly:

```Javascript
Rx.Observer.create (function(emittedValue){
        document.getElementById('subscriptionTwoOutput').innerHTML += emittedValue + ', ';
    }, // onNext()
    {}, // onError()
    function(){
        document.getElementById('subscriptionTwoOutput').innerHTML += '... done!';
    }); // onCompleted()

source.subscribe(observer); 
```

The `.onError()` and `.onCompleted()` both pass information to the observer, but once they fire the Observable does not emit any more data.  The observer does not need to define handlers for these events.


You can try this out in this fiddle:

### [jsfiddle.net/brnvndr/eakrb004/](https://jsfiddle.net/brnvndr/eakrb004/)


Observables can be created automatically from many sources:

- `Rx.Observable.from()` - from an array or collection
- `Rx.Observable.fromEvent(<target control>, <event name>)` - from a DOM event
- `Rx.Observable.fromPromise()` - even from promises

Additionally, Observables define methods for manipulating the data returned to the observer:

- `.map( x=> 10 * x)` - transform data.  this is very useful for parsing HTTP response to get the data payload
- `.filter(x => x > 2)` - limit data returned

A good place to start reviewing Observables is in this [online book](https://xgrommx.github.io/rx-book/), but be warned, this is a very deep topic.  You don't need to understand all of this to use Observables in Angular, but understanding Observables outside the context of Angular will enable you to use them more effectively.  


