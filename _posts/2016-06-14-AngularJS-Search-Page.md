---
title: 'AngularJS Search Page'
date: 2016-06-14T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/06/14/AngularJS-Search-Page/
categories:
  - Development
  - Projects
tags:
  - AngularJS
  - HTML
  - JavaScript
  - CSS
---


This article demonstrates a basic technique for a search page in Angular.
I've created a **[demo page](https://jsfiddle.net/brnvndr/11asbstj/)**  using JSFiddle that implements the following:

- Angular databinding to a repeating list
- Using a static JSON object
- Creating a service for common data
- Angular filtering
- Using the angular class directive to apply dynamic CSS

This search page uses an all-at-once search technique where the application loads the entire data set first, then filters the results.  This is a simplistic technique, which works for up to a few thousand records.   The advantage is that once loaded, the data doesn't need to be re-loaded when the filters change, which can make the search appear very quick to the user.  I do not demonstrate any paging techniques here, and in my opinion, paging is something that needs to go away.   Paging as a technique is largely a trick for reducing page bandwidth, and while I agree you can't just load 10K items onto a page, it seems like many places where paging is used have < 1K records.  As a user paging isn't something anyone really wants.   If someone sent you a spreadsheet of customers, would you rather have the list separated into multiple worksheets or one large list?  Yet many sites still force the user thru multiple pages to get what they are looking for.  Thankfully more sites now are starting to use an "infinite" scrolling technique, but that is out of scope for this article.

## Databinding 

The data for the search page is stored in a static JSON collection in the main javascript file.  In a production application, this would be retrieved by a service call to some API endpoint, but for testing, you will also likely have this as a separate JSON file that can be loaded for your unit tests.  This is an extremely powerful technique.  It allows us to build out the front-end before the back-end is available, which allows us to prototype and iterate much faster.  Even after the back-end is available, the static JSON is still useful for unit testing.   

To make the JSON available to the view, we create the controller: `searchController` and set the directive: `ng-controller="searchController"` in the view. This allows every child element to have access to the scope of this controller.  For simple databinding, we either use the {% raw %}`{{}}`{% endraw %} binding expression or set the `ng-model` directive, but since we have a list of items we want to databind, we need to use the `ng-repeat` directive:

{% raw %}
```html
    <tr ng-repeat="partNumber in searchResults">
        <td>{{partNumber.modelName}}</td>
        <td>{{partNumber.partNumberName}}</td>
        <td>{{partNumber.inventoryPartNumber}}</td>
        <td>{{partNumber.manufacturerPartNumber}}</td>
        <td>{{partNumber.partNumberStatus}}</td>
        <td>{{partNumber.modelCategoryName}}</td>
        <td>{{partNumber.partNumberListPrice}}</td>
    </tr>
```
{% endraw %}

The `ng-repeat` directive will iterate through each JSON object in the `searchResults` collection, and bind the {% raw %}`{{partNumber.XXX}}`{% endraw %} expressions.  

## Simple Filter

Adding a single text box filter is quite simple:

```html
<input type="Text" ng-model="query" />

<tr ng-repeat="partNumber in searchResults | filter:query>
```

Each time the value in the `<input>` changes, angular re-binds the list, checking for the value of query in any of the string properties of `partNumber`, so you can essentially search every column with a single text box.  If instead we want to search just a single field, we can set the filter expression as follows:

```html
filter:{partNumberName:query}
```

## Multi-Field Filter

This page also demonstrates multiple search fields, including "does not contain" fields.   The neat thing is that you can specify the JSON property directly in the `<input>`, and still keep the filter expression simple:

```html
<input type="text" id="modelContains" ng-model="query.modelName" class="includeInput"/>
<input type="text" id="nameContains" ng-model="query.partNumberName" class="includeInput"/>
```

Unfortunately the exclude filter is more complicated.  With includes, a blank query value essentially matches everything, so by negating the blank value, you are filtering out everything! I found I needed to supply conditional logic to only apply the filter if the `<input>` contained a value.   Additionally, you need to specify each filter field separately, so it is not as clean as the includes:

```html
<tr ng-repeat="partNumber in searchResults | filter:query | 
            filter: (exclude.modelName.length > 0 ? {modelName: '!' + exclude.modelName} : '') |
            filter: (exclude.partNumberName.length > 0 ? {partNumberName: '!' + exclude.partNumberName} : '') |
            filter: (exclude.inventoryPartNumber.length > 0 ? {inventoryPartNumber: '!' + exclude.inventoryPartNumber} : '') |
            filter: (exclude.manufacturerPartNumber.length > 0 ? {manufacturerPartNumber: '!' + exclude.manufacturerPartNumber} : '') |
            filter: (exclude.partNumberListPrice.length > 0 ? {partNumberListPrice: '!' + exclude.partNumberListPrice} : '')">
```

## Services

The advanced search mode contains two dropdowns for status and category. These come from the `lookup` service, which highlights how Angular wants you to think about separation of concerns.   As you can see in the `searchController`, we just store the references to the lists retrieved from the `lookup` service - but we could just as well add these lists directly to the controller. But what happens if we want to include the category lists in another view?  We would need to add the list to that controller was well - not good.   By using the service, we can inject `lookup` everywhere it needs to be used.   Additionally, at some point we may want to retrieve the category list from a database.   A controller should absolutely not make any back-end calls to an API or database, as this involves configuration details that we won't know until runtime.   The controller's job is to expose data and logic to the view - that's it.   It should be irrelevant to the controller where the data comes from.   

## Angular Class Directive

Lastly, I demonstrate using the ng-class directive to apply dynamic CSS to the view.  In this example, we have two search modes - basic and advanced.  We need to hide components depending on the mode.  First off, we do not want to do this in the controller!  The controller should not touch the DOM at all - it should only operate on and communicate through the scope.   Secondly, we want to use CSS classes instead of iterating through DOM elements. Here's how the operation should look like:

1. User changes search mode by clicking on link: `<a href="#" ng-click="toggleSearch('advancedSearch')">Advanced Search</a>`
2. `searchController` updates the current searchMode, which corresponds to the CSS class name to activate:

```javascript
$scope.toggleSearch = function(searchMode) {
         $scope.searchMode = searchMode;
         $scope.query = '';
         $scope.exclude = '';
     }
```

3. The three main view elements: `searchHeader`, `searchFilter`, `searchResults` have an `ng-class` directive, which can apply a CSS class.  When `$scope.searchMode` changes, the classes for all three will be updated.   Here is what the classes look like for the `searchResults`.  In basic mode, this div takes up the full row from left to right, but when `advancedSearch` is active, we need to shift the div over

```javascript
.searchResults.advancedSearch {
  margin-left:250px;
}
.searchResults.basicSearch {
  margin-left:0px;
}
```

The power of this technique is that any time we want to add another UI component that behaves differently with the different modes, we merely need to define the appropriate CSS classes for the new component.

## Conclusion

While simple, this example provides a basic template for setting up a search page.
