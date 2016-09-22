---
title: 'Javascript Promises'
date: 2016-09-21T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/09/21/Javascript-Promises/
categories:
  - Research
tags:
  - JavaScript
  - Promise
---

Learning a new framework like Angular2 brings up a large number of technical items that you end up using, but not necessarily understanding right away.  For example, a simple DataService had me implement the following code: 

```javascript
getModels(): Promise<Model[]> {
    return Promise.resolve(MODELS);
}
```

Even though this is a mock object, it is good practice to assume that data operations may take some time - enough where async operations come into play.  

Wanting to understand a bit more, I did some quick research and a demo on promises.  Here's what I found out: much of this from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

- Promises are a newer language feature added to JavaScript in 2013.
- Promises are used for asynchronous operations.
- The **Executor** is a function that receives function references `resolve` and `reject`
- The Executor is called immediately.  At some point in this function, either the `resolve` or `reject` functions are be called.
- There are three states to a promise: **Pending**, **Fulfilled**, **Rejected**
- When `resolve` is called, the `then` function is called on the promise.  The promise is considered **fulfilled**
- when `reject` is called, the `catch` function is called on the promise, and is considered **Rejected**
- promises can be chained.  This allows for a degree of synchronization betweens asynchronous operations.  
- note that `then` always returns a promise, whether one is explicitly created or not. The implicit promise essentially resolves immediately

## Basic promise Format

``` javascript
var promise = new Promise(
    function(resolve, reject) {
        //some sort of long running / async operation
        if (true) {
            resolve("success");
        }
        else {
            reject("failure");
        }
    }
);
promise.then( // called when resolve() executed
    function(result) {
        console.log("Success: " + result);
    })
.catch( //called when reject() executed
    function(reason) {
        console.log("Failure: " + reason );
    });
```

Here is an example JS Fiddle I wrote that demonstrates promises and promise chaining:

### [jsfiddle.net/brnvndr/Lep6dghu/](https://jsfiddle.net/brnvndr/Lep6dghu/)
   