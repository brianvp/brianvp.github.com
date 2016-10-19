---
title: 'Angular2 Testing Introduction'
date: 2016-10-11T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/10/11/Angular2-Testing-Introduction/
categories:
  - Development
  - Projects
tags:
  - Angular2
  - Jasmine
  - Unit Testing
---

Looking at unit testing in Angular2, following the offical [docs](https://angular.io/docs/ts/latest/guide/testing.html).   Unit Testing is a large topic, so this is just an initial look and summary of what I've learned so far.  

## Why Unit Testing

According to the documentation, Unit Testing as three main purposes:

1. Guard against regressions in code.  Changing code in an established application is frequently a source of bugs, and having a battery of tests available can alert the developer to issues quickly (assuming there is a test that covers that particular issue)
2. Describe how services, pipes, and components should operate.   Even if you are good at reading code, it is not always easy to visualize what inputs and outputs should look like.  
3. Expose design flaws in component design.  If it is hard to write a test for a component, it is a strong sign that the component has design flaws, such as doing too many things, containing unnecessary dependencies, etc.  

## Jasmine

[Jasmine](http://jasmine.github.io/) is a unit testing framework for JavaScript that the Angular team recommends using.

- In Jasmine, a test is known as a "spec".   
- Specs are stored in a `.spec.ts` file.  Each component/service/pipe that has tests should have their own spec file, e.g. `App.component.spec.ts`. 
- specs are defined with the global Jasmine function `it()`
- Multiple specs for the same component are known as a "suite".  A suite is defined with the global jasmine function `describe()`
- The purpose of a spec is to define an assertion about what a component should do, or be.  An assertion is either true or false.  for example, an assertion may be: When we pass 2 to the function square(), the result will be 4
- To create an assertion in Jasmine, use the `expect()` global function, inside an `it()` function.  
- Jasmine defines a large number of "Matchers" to be used with `expect()`
    - toBe()
    - toEqual()
    - toBeUndefined()
    - toBeTruthy()
    - toContain()    
    - etc.

### Example Test

```javascript
describe('Math Tests', () => {
    it('Math.pow(2,32) should be 4294967296', () => {
        expect(Math.pow(2,32)).toBe(4294967296);
    });

    it ('Math.sqrt(64) should be 8', () => {
        expect(Math.sqrt(64)).toBe(8);
    });
});

```

## Karma

Jasmine tests need to be run to see if they succeed or fail.  [Karma](https://karma-runner.github.io/1.0/index.html) is a unit test runner the Angular team recommends.  Karma runs via the node command: `npm test`.  It launches a browser instance that hosts the tests, and can be configured to run all tests automatically after application code is updated.

After each test run, the results will output in a command window (not the browser).  While a bit verbose, you will be able to clearly see if any tests have failed.  

I'm not using karma for the demo code below, but it is a part of the [angular-cli](https://cli.angular.io/) environment  

## Jasmine + Angular

By itself, Jasmine knows nothing about an Angular application, so we need to use an Angular test infrastructure to make useful unit tests. 

- Need to import all components/services that the test/spec(s) will be using
- use the global Jasmine function `beforeEach()` to initialize components.  `beforeEach()` runs before every individual `it()` execution.
- some items such as pipes and services do not depend on the angular environment running, and can be tested with little setup in Jasmine, besides instantiating the class.
- the primary class for interacting with the Angular environment is the `TestBed` class. This class is responsible for:
    - Instantiating the component that will be tested via `TestBed.createComponent()`. this will return a special `ComponentFixture` class that contains the component instance as well as a few other testing properties
    - configuring the angular environment via `TestBed.configureTestingModule()`
    - querying the components template + rendered DOM via `debugElement` and `nativeElement` properties of the `ComponentFixture`. Note that these elements are hierarchical i.e. `debugElement` can contain one or more child `debugElement`s
    - provide access to the component's properties & methods via `fixture.componentInstance`.  Note that the properties will not be defined until `fixture.detectChanges()` is run.  

### Example Component Test

```javascript
import { TestBed, ComponentFixture } from '@angular/core/testing';
import { By }           from '@angular/platform-browser';
import { DebugElement } from '@angular/core';
import {SampleComponent } from './sample.component';

let comp:    SampleComponent;
let fixture: ComponentFixture<SampleComponnent>;

describe('SampleComponent', () => { 
  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [ SampleComponent ], 
    });

    fixture = TestBed.createComponent(SampleComponent);
    comp = fixture.componentInstance;
  });
  
  it ('Should create the component', () => {
    let app = fixture.debugElement.componentInstance; 
    expect(app).toBeTruthy();
  });
  
  it ('H1 title should be: Sample Component', () => {
    let de = fixture.debugElement.query(By.css('h1'));
    expect(de.nativeElement.textContent).toEqual("Sample Component");
  });
  
  it ('component property should be 25', () => {
    fixture.detectChanges(); // the component does not initialize until detect changes happens
    expect(comp.componentProperty).toEqual(25);
  });
  
  it ('component component property should be undefined before initialization', () => {
    expect(comp.componentProperty).toBeFalsy();
  });
});
```

### Demo

<iframe src="https://embed.plnkr.co/DzLg9W8OyVgNGA2M3qWq/" width="100%" height="800"></iframe>
