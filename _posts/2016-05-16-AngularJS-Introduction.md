---
title: 'AngularJS Introduction'
date: 2016-05-16T00:00:00+00:00
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


AngularJS is a JavaScript framework developed by Google for front-end web development. It allows creation of a class of applications known as "Single-Page-Applications" (SPA). A SPA typically loads all required code (HMTL/CSS/JavaScript) in a single page request.  In response to user actions, additional resources are loaded as necessary thru XMLHttpRequests / AJAX.  There is no postback or transfer of control to another page - to the browser the user is simply sitting on / interacting with a single page.  This architecture allows for a fluid, seamless feel to a web application, bringing it closer to what a native application feels like.  While network latency is often a limiting factor in performance, traditional web applications exacerbate this by constantly loading/reloading application components, and processing presentation details on the server.  In a SPA, shared components are typically loaded once, with page-specific content loading dynamically as needed(often only a few kilobytes).  


AngularJS was written specifically for data-oriented applications, which most often refer to line of business applications (LOB).  Realizing that a common need was writing data entry forms hooked up to databases, the Angular architecture was optimized for performing data binding operations. Additionally, Angular philosophy stresses strict separation of concerns and testability.  It's no secret that web applications (and business applications in general) are often bloated and buggy,and some of this can be blamed by poor platform architecture.  Obviously no tool is going to make up for a poor development approach, but a goal of Angular is to make it easier to write better software for those that care about such things...

## An Application Example

Let's see what an angular application looks like.  At the most basic level an application will look something like this:

* References to the AngularJS library
* an HTML tag with the ng-app attribute / directive
* a definition for the main application module
* html markup with angular bindings  

```html
<!DOCTYPE html>
<html ng-app="myApp">
  <head>
  </head>

  <body>
	<div ng-controller="defaultController">
	  <h1>{{greeting}}</h1>
	</div>

	<script src='https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.2/angular.min.js'></script>
	<script>
		var app = angular.module('myApp', []);

		app.controller('defaultController', function($scope){
		  
		  $scope.greeting = "Hello, World!";
		  
		});
	</script>
	</body>
</html>

```

The main components of this application are:

1. the html file itself, usually index.html.  This is the primary page the browser loads.
2. Angular library reference
3. A script block (usually in a separate file i.e. app.js) that defines the main application module.
4. A script block that defines a "controller".  The job of the controller is to provide data to our application
5. HTML markup containing data binding(s) to the model (defined in the controller).  

When this page loads, the application module "myApp" is instantiated in the current browser context, and given a controller reference.  Then, the HTML markup 
is processed to reflect any data changes.  It's important to realize at this point that the page is already "loaded" as far as the browser is concerned. 
Initially the section `<h1>{{greeting}}</h1>` is rendered as `{{greeting}}` by the browser.  When Angular runs, it sees this as a databinding, and replaces 
this with whatever value the `greeting` attribute is on the current scope.  

One implication of this is that you can have an Angular "application" pretty much anywhere you can serve an HTML page.  For example, you could take a legacy
ASP.NET application and replace the search page with an embedded angular application, or you could create an entire website composed of a collection of Angular
applications.   


## Angular Terms

Like any framework, Angular has a number of key terms you should be familiar with:

* **Template** - a partial block of HTML that contains HTML and Angular bindings
* **View** - The rendered template displayed in the browser - a template becomes a view after Angular replaces the bindings with values
* **Model** - The model represents the data that is available to use on the current page.  
* **Scope** - All angular components access the Model through the scope.  What is available in the scope depends on where the element is on the current page.
* **Controller** - A JavaScript function that provides data and behaviors to a view.  The controller should not engage in any DOM manipulation.
* **Module** - A module is a container for the various parts of your application - Defined components must reference a specific module
* **Directive** - Directive are HTML attributes that extend the tag with additional functionality, or represent a new tag altogether.
* **Service** - Represents common functionality shared by all parts of an application.  
* **Two-Way Databinding** - The concept that changes to a property of the scope are immediately available to the directive/view as well as the controller. 

## Angular Concepts

Angular is an opinionated framework - it was designed with certain principles in mind, and it strongly encourages you to follow those.  

### Separation of Concerns

In Angular, it is very important that components do one thing well, and do not cross prescribed boundaries.  

* Controllers can manipulate the model, add data and behaviors to the scope, but should not directly update the DOM
* Directives and templates should interact with the model through the properties exposed on the scope.  Complex interactions should be handled at the
controller level
* Use Filters to format and massage data, rather than JavaScript blocks embedded in the template
* Break common template code into directives e.g. Customer Address
* Take common controller code and place into a service e.g. a list of US states

### Dependency Injection

Angular uses the dependency injection pattern to pass services to controllers, filters, directives, etc.  Controllers should never instantiate service objects directly. By following this pattern, you will ensure that your application components are as testable and as simple as possible.  For example, having your controller open a connection to a service endpoint or database is considered bad practice.  Where are you getting your server / url from?  What if we want to hit the staging server instead of production?  

## Resources

* [Official Angular Tutorial](https://docs.angularjs.org/tutorial)
* [Official Developer Guide](https://docs.angularjs.org/guide)
* [Official API Docs](https://docs.angularjs.org/api)
* [GitHub Project](https://github.com/angular)
* [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection)
* [Angular Style Guide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md)


## Conclusion

This is the first of many posts exploring AngularJS.  I'll be going through the requirements for using AngularJS to create a typical line of business application. It should be note that at the time of this writing (May 2016), Angular is going through a large conceptual shift from 1.X to 2.X.   I will be focusing on concepts related to 1.X, primarily as this is what I'm using to build production applications *now*. 


