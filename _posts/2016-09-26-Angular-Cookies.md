---
title: 'Angular Cookies'
date: 2016-09-26T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/09/26/Angular-Cookies/
categories:
  - Research
tags:
  - Angular
  - Angular2
  - Cookie
---

Looking at adding cookies to an Angular2 demo the other day, and found out the [$cookie](https://docs.angularjs.org/api/ngCookies/service/$cookies) service is not implemented natively in Angular2.  Doing some quick googling, found a decent implementation: [https://github.com/salemdar/angular2-cookie ](github.com/salemdar/angular2-cookie ).  

To use, you simply need to import the `CookieService` inside of `app.module`, making it available to other components.  

Interacting with the cookies is pretty simple:

### Setting
```javascript
    this._cookieService.put("cookieSimple", "A Simple Cookie");
    this._cookieService.putObject("cookieObject", {option1:true, option2:1, option3:"default"});
    this._cookieService.put("cookieWithOptions", "Expires in a long time", {expires:"Mon, 30 Jun 2290 00:00:00 GMT"});
```


### Getting
```javascript
    this.cookieSimpleValue = this._cookieService.get("cookieSimple");
    this.cookieObjectValue = this._cookieService.get("cookieObject"); // or getObject()
    this.cookieWithOptionsValue = this._cookieService.get("cookieWithOptions");
```

plunker demo:

### [plnkr.co/edit/k1KCVPupsdiSk2gvkKK1](https://plnkr.co/edit/k1KCVPupsdiSk2gvkKK1)