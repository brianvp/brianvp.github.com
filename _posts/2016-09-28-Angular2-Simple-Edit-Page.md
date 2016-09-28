---
title: 'Angular2 Simple Edit Page'
date: 2016-09-28T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/09/28/Angular2-Simple-Edit-Page/
categories:
  - Development
  - Projects
tags:
  - AngularJS
  - Angular2
  - HTML
  - JavaScript
  - CSS
---

Going over the Angular2 docs on forms, and decided to bring forward my 1.x [simple forms demo](https://jsfiddle.net/brnvndr/44Let626/)

It ports over fairly well, but a significant number of changes needed to be made.  Honestly it seems like it would take an enormous effort to convert an existing 1.x application to Angular2.  

### Specific Changes

- update directives
    - `ng-model` -> `[(ngModel)]`
    - `ng-class` -> `[ngClass]`
    - `ng-submit` -> `(ngSubmit)`
    - `ng-show` -> `[hidden]` (could have also used `*ngIf`)
    - `ng-maxlength` -> `[maxlength]`
    - `ng-disabled` -> `[disabled]`
- instead of defining a telephone `filter`, I created a `pipe`
- Create `ManufacturerService` with a `type` class and a `mock`
- Create generic `LookupService` for the list of states
- `angular.copy()` does not have a direct replacement in Angular2.  `Object.assign()` works well as long as a deep copy is not needed, which isn't the case here (Manufacturer doesn't contain collections, just simple types)
- instead of defining an empty manufacturer JSON object, we can instantiate a new, empty manufacturer. (to be used when the `new` button is clicked)
- The `currency` filter will not work with an empty string.  In edit mode, blanking out the `<input>` causes {% raw %}`<span class="displayValue" [ngClass]="pageMode">{{manufacturer.creditLimit | currency }}</span>`{% endraw %} to raise an exception.  The workaround is to set the `<input type="number"`
    - alternately, one could argue that the view controls should not be bound to the same object the edit controls are bound to
- I wasn't able to access the `manufacturerForm` properly from the `simple-edit-form.component.ts`.  
    - within the template, `manufacturerForm.valid` is available, but in the `component.ts` it is undefined 
    - ended up using `manufacturerForm.checkValidity()`
    - this [plunker](http://plnkr.co/edit/IElMhx2Kcos7VLrI2QfC?p=preview)  (not mine) shows the forms `valid` property as accessible in `component.ts`, but I didn't get this to work  
- needed to set `#manufacturerForm="ngForm"` inside the `<form>`
- In addition to the `id`, `name` attributes, needed to set `#nameInput="ngModel"` inside the `<input>`'s using validation. 
- spent a lot of time figuring out why `nameInput.errors.required` was giving undefined errors.  It looks like you always need to check for `valid` first, as the `errors` collection is only defined when errors are present: `[hidden]="nameInput.valid || !nameInput.errors.required"` The `||` causes a short-circuit evaluation in the expression.  If you remove the `nameInput.valid` from the example below you can repro this issue.


### Demo

<iframe src="https://embed.plnkr.co/ZwQB5EENaR7wYdMA2rDs/" width="100%" height="800"></iframe>

