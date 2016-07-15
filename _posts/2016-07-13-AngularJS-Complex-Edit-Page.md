---
title: 'AngularJS Complex Edit Page'
date: 2016-07-13T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/07/13/AngularJS-Complex-Edit-Page/
categories:
  - Development
  - Projects
tags:
  - AngularJS
  - HTML
  - JavaScript
  - CSS
---


This article demonstrates a "complex" edit page in Angular.
I've created a **[demo page](https://jsfiddle.net/brnvndr/vzwt1Ly3)** using JSFiddle.

So what makes an editing page "complex"? In my experience complexity begins whenever you have two or more non-lookup entities on a single page or view.  This is primarily due to issues regarding state, foreign key management, and additional bindings.  The most common scenario is with the parent-child (master/detail) relationship.   In this example I model a common product pattern with model-sku.  For many products it is typical to have a basic model that comes in a number of different configurations and sizes. It is important that we can uniquely identify a specific product size for sales, shipping, and inventory purposes.  At the same time, we want to group related part numbers together under a common "model" so that they can share descriptions, images, pricing, categories, etc.  

Other common parent-child relationships

- Sales Header -> Sales Detail
- Purchase Order Header -> PO Detail
- Formula / Assembly -> Bill of Materials / Recipe Items
- Employee -> Time / Punch/ Hours history
- Location/department -> Employees
- Vendor -> Vendor Contacts

## Page Structure

The core structure of the page is similar to the **[Simple Edit Page]({{site.baseurl}}/2016/06/14/AngularJS-Search-Page/)**.  The flexiblity of JSON objects to contain arrays of objects makes binding much simpler.  We simply need a reference to `$scope.model`, and optionally `$scope.modelPrior`.  The part numbers are stored as an array of objects here `$scope.model.partNumbers`.   Binding this inside the view is simple, via `ng-repeat="partNumber" in model.partNumbers"` Once again the beauty of this approach is that we can operate on `model` without direct knowledge of how the view renders each part number.  For example, `deletePartNumber()` simply removes a part number from the array.  It doesn't need to do a DOM lookup for that row and remove markup - we just adjust the data and angular re-binds the table (instantly I might add).  

There is some work to do with managing the detail records.

1. We need to add an empty line at the bottom for adding an additional record
2. We need to assign temporary key values to new items
3. We need a way to remove records and keep track of removed records


The most important principle with parent-child relationships is that all changes made should be atomic.  A simplistic system might implement separate db/service calls for editing individual records, but this doesn't allow the user to makes changes, then change their minds and cancel.  It also creates the problem of partial updates because of system issues.  What if the user enters invalid data into the 20th row of a detail screen, causing the system to crash after processing the previous 19 records?   With Angular data binding this is pretty easy to avoid, at least at the client level (the service layer still needs to consider transactions). 

## Adding Detail Records

There are two general approaches for creating a new line row.  First, we can simply add a new, empty partNumber to the `model.partNumbers` collection.   I don't like that approach however, as it complicates the binding in the `ng-repeat` section.  Additionally, when we go to persist the model, we will either need to strip out the blank row, or the service layer will need to check for blank values. The second approach is to add a `<tr>` below the `ng-repeat`, and create a placeholder object for binding, `newPartNumber`.  On page load or when the user clicks the add button, newPartNumber is loaded with blank values.   

It is also important to assign a temporary key to the part number.  This serves two purposes. One, it signals to the business layer that these are records that need to be inserted, and secondly it gives us a handle for removing the new record. 

## Removing Detail Records

As mentioned earlier, removing a part number is as simple as removing it from the array.  To do this, we place a delete button on each row.  The delete button will reference the `deletePartNumber()` function.  Angular binding is quite elegant here. Each `ng-click` references the part number to delete via databinding:

```html
<button class="editButton" ng-class="pageMode" ng-click="deletePartNumber(partNumber.partNumberId)" ng-show="partNumber.flagCanDelete === true">Delete</button>
```

However, we should also consider when it is appropriate to remove a part number.  A good rule of thumb is that an entity can be removed if it hasn't been used as part of another relationship - i.e. the part number has not been sold, purchased, etc.  To simulate this, I've marked some of the part numbers as not eligible for deletion.  The `ng-show` above takes advantage of this and hides the delete button.  Remember that the service layer should also enforce this rule - this is just a UI convenience.  

## Validation

Both the model and part number areas are part of the same `<form>`, the advantage being that the save operation can check a single global `$valid` state.  For this example I've put `required` attributes on several of the part number fields.  When these are empty, the form state is updated to be invalid, which in turn disables the save operation using `ng-disabled`

## Conclusion

I was pleasantly surprised to see how straightforward a parent-child relationship can be implemented in Angular.   