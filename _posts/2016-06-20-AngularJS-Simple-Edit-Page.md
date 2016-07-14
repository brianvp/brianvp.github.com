---
title: 'AngularJS Simple Edit Page'
date: 2016-06-20T00:00:00+00:00
author: brianvp
layout: post
categories:
  - Development
  - Projects
tags:
  - AngularJS
  - HTML
  - JavaScript
  - CSS
---

This article demonstrates a simple edit page in Angular.
I've created a **[demo page](https://jsfiddle.net/brnvndr/44Let626/)** using JSFiddle that implements the following:

- Binding a single entity to a set of HTML form inputs
- View and Edit modes - hide the input fields in view mode
- Buffering model values so that changes can be discarded if necessary
- Using filters to format display values
- Basic validation techniques
- Use of ng-submit vs ng-click

## Databinding

Databinding is very straightforward.  We store the current entity data in `$scope.manufacturer`, and then either set `ng-model` on the inputs or use the databinding `{{}}` markup.  Changes to the inputs are (by default) immediately reflected in the model, and likewise changes to the model are immediately reflected in the inputs and other bindings. For the state dropdown, we use the `lookup` service to retrieve and bind the list of states. 

A common pattern for data entry pages is to have both a view mode and edit mode. This is accomplished with a set of dynamic css classes on each of the related controls. To hide controls, we set `display:none`, which allows the control to be in the DOM but take up no space, as opposed to `display:hidden` 

Most data entry pages will also contain the common CRUD operations - which we implement here except for deletion in the `manufacturerController`:

- `editManufacturer()`
- `newManufacturer()`
- `saveManufacturer()`
- `cancelEdit()`

These functions are responsible for setting the current editing mode, and managing the `manufacturer` buffer / binding. We don't interact with the DOM except through the model.  Notice in `saveManufacturer()` we check if the `manufacturerForm` is valid before saving.  The form and its collection of bound values are added to the $scope by Angular, so we do not have to look up in the DOM.   Last, look at the use of `angular.copy()`.  Remember that `$scope.manufacturer` contains a reference to a JSON object.  If we want to revert it back to the original values, we need to *copy* the values from `$scope.manufacturerPrior`. If instead, we set `$scope.manufacturer = $scope.manufacturerPrior`, we will have two members pointing to the same object, which isn't what we intended.  There will be a subtle error if we go in and out of edit mode.  The first time, it will work as expected, since there are two separate objects, but the second time, the form controls will be bound to the same reference that `$scope.maufacturerPrior` holds!

## Filters

In angular, filters are used to manipulate displayed data, either to supress data or to format data.  In this example we use a filter to format currency and phone numbers.  The currency filter is quite simple, simply specify the filter expression in the binding:

```html

<span class="displayValue" ng-class="pageMode">{{manufacturer.creditLimit | currency}}</span>

```

Currency is one of a few default filters available in Angular. Others include:

- number
- date
- json
- lowercase
- uppercase

Unfortunately these filters don't work on input fields. To format data inside an input we will need to resort to custom javascript or find an appropriate library control.  

We can also create custom filters, which I've done here for formatting telephone number using a sample filter from [Stackoverflow]( http://stackoverflow.com/a/12728924/24892).  To do this you call the filter function on the main app module:

```javascript
productManagerApp.filter("telephone", function () {
    return function(telephone){
        if (!telephone)
            return '';
        var value = telephone.toString().trim().replace(/^\+/, '');

        if (value.match(/[^0-9]/)) {
            return telephone;
        }

        ...

        return (country + " (" + city + ") " + number).trim();
    };
});

```

Here you specify the name of the filter `telephone`, then define a function that takes an input (bound data flowing through the filter) that returns a string output, in the desired display format.  Once this is defined, the filter can be used in bindings as follows:

```html
<span class="displayValue" ng-class="pageMode">{{manufacturer.faxNumber | telephone}}</span>
```

This allows for very clean usage and keeps the formatting logic out of the controller and view.  

## Forms and Validation

The edit fields are contained in an HTML form.   Normally form submission causes a page postback to the server, but angular supresses this by default, unless the `action` attribute is specified in the form.  Strictly speaking you can develop without forms in angular, but it is still helpful to do so, primarily for validation.  Forms and their associated named controls have the following properties defined that can be programmatically accessed:

- $pristine - set to true if user has not interacted
- $dirty - set to true if user has interacted
- $valid - true if the form/control is valid
- $invalid - true if the form/control is invalid
- $error - a hash that contains references to invalid items

This allows us to check `manufacturerForm.$valid` or `manufacturerForm.nameInput.$valid`.   

To submit the form, we set the `ng-submit` attribute of the form to `saveManufacturer()`. Interestingly we can simply change the `save` button to specify `ng-click()` and call the function directly.   However it is better to use the form submission model if possible.  Additionally the angular docs [warns](https://docs.angularjs.org/api/ng/directive/ngSubmit) about using `ng-submit` and `ng-click` in the same context.  

To prevent saving when validation fails, we use the `ng-disabled` attribute on the input button:

```html
<button type="submit" class="editButton" ng-class="pageMode" form="manufacturerForm"  ng-disabled="manufacturerForm.$invalid">Save</button>
```

This will visually disable the button as well as the `enter` submit action.  It is also a good idea to check the valid status in the controller function as well

```javascript
$scope.saveManufacturer = function () {
    if ($scope.manufacturerForm.$valid) {
        $scope.manufacturerPrior = {};
        $scope.pageMode = "viewMode";
    }
}
```

A  word on button layout - for the `ng-submit` method to work, the button / input must be inside the form.  This is not ideal for layout purposes, as you may want to keep all the action buttons together.  However in this example, `ng-click` does not fire for the edit button while inside the form!  There are a few workarounds for this.  In newer browsers, you can use the `form` attribute on a button to trigger a submit from outside the form, but this does *not* work on IE11.  These CRUD functions are too important to not work on IE, so that's out.  There are also a number of ways to trigger the submit via JavaScript, but I wasn't able to find a reliable/simple one, and is outside the scope of this article anyway.

There are two types of validation on the page.  The first uses the `required` attribute, which is fairly straightforward.  To alert the user when they need to enter a required field, we place an error `span` after the form field, using the `ng-show` attribute:

```html
<span class="error"  ng-show="manufacturerForm.nameInput.$error.required">Name field is required</span>
```   

By default an HTML5 form will display a warning popup on save, but since we are including custom messages here we can supress this by setting the `novalidate` attribute on the form.  

The second type of valiation is a `minlength` and `maxlength` attribute on the account number field.  There are two ways to set this, use either the `minlength` or the `ng-minlength` syntax.   The difference I noticed was that with `maxlength`, you physically couldn't type more into the field, whereas with `ng-maxlength` you were able to type unlimited chars (but the validation still failed if above the max length) 

## Conclusion

This article demonstrated a simple technique for a data entry form.  Angular does an impressive job minimizing the amount of code in the controller needed to support the view and edit behaviors. 