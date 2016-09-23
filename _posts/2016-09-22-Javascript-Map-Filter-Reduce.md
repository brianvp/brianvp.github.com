---
title: 'Javascript Map Filter Reduce'
date: 2016-09-22T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/09/22/Javascript-Map-Filter-Reduce/
categories:
  - Research
tags:
  - JavaScript
  - Functional Programming
---

Today I found myself needing to brush up on some functional programming concepts in JavaScript, namely the `map()`, `filter()`, and `reduce()` array methods. I found a good [primer](http://cryto.net/~joepie91/blog/2015/05/04/functional-programming-in-javascript-map-filter-reduce/) and went through the examples.

## Map Operation

- used to modify the input set of array values into another output array
- does not change the values of the input array
- map function should only modify the input parameter - should not modify values outside of the map function

```javascript
var namesMapMethod = names.map(function(name){
    return name.toUpperCase();
});

```

additionally, you can change multiple .map() functions together

```javascript
var namesChained = names.map(function(name){
    return name.toUpperCase();
}).map(function(name){
    return name.replace('A', '@'); 
});
```

## Filter Operation

- used to return a subset of array values from the input array
- does not modify the input array
- should not modify any values outside of the filter function

```javascript
var filtered = names.filter(function(name){
    return (name.indexOf('a') > -1);
});

```

## Reduce Operation

- takes an input array and returns a scalar result (unless you define the result as an array)
- can be used for string concatenation, calculating totals, etc.


```javascript
var nameString = names.reduce(function(result, name){
    return result + name + "-";
}, "");
```

demo:

### [jsfiddle.net/brnvndr/94rj8nhb/](https://jsfiddle.net/brnvndr/94rj8nhb/)
   