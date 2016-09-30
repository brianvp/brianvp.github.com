---
title: 'Angular InMemoryDataService'
date: 2016-09-30T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/09/30/Angular-InMemoryDataService/
categories:
  - Research
tags:
  - Angular2
  - Web API
  - InMemoryDataService
---

Reviewing [the HTTP Angular Tutorial](https://angular.io/docs/ts/latest/tutorial/toh-pt6.html) this morning, I was really confused how the sample "Web API" was working in the sample plunker.  

It started out as:

```javascript
search(term: string): Observable<Hero[]> {
    return this.http
               .get(`app/heroes/?name=${term}`)
               .map((r: Response) => r.json().data as Hero[]);
```

`app/heroes/?name=${term}` is clearly the API call, but where is the function that processes it?  I assumed that this was in HeroesComponent, since `app.routing.ts` defines the `heroes` route.

```javascript
{
    path: 'heroes',
    component: HeroesComponent
  }
```

But why would the API be in a component class, that makes no sense! Moreover, the only likely function,

```javascript
  getHeroes(): void {
    this.heroService
        .getHeroes()
        .then(heroes => this.heroes = heroes);
  }
```

doesn't include any parameters, and a `console.log()` very clearly showed this function is not called during search operations.  

I was close to just assuming there was some invisible api I couldn't see, when I re-read the tutorial and noticed something about the `InMemoryDataService`:

```javascript
import { InMemoryDbService } from 'angular-in-memory-web-api';
export class InMemoryDataService implements InMemoryDbService {
  createDb() {
    let heroes = [
      {id: 11, name: 'Mr. Nice'},
      {id: 12, name: 'Narco'},
      {id: 13, name: 'Bombasto'},
      {id: 14, name: 'Celeritas'},
      {id: 15, name: 'Magneta'},
      {id: 16, name: 'RubberMan'},
      {id: 17, name: 'Dynama'},
      {id: 18, name: 'Dr IQ'},
      {id: 19, name: 'Magma'},
      {id: 20, name: 'Tornado'}
    ];
    return {heroes};
  }
}
```

the return name is `{heroes}`.  What if I changed that to `{heroesx}`?  Sure enough, the `app/heroes` call failed.   I then tried changing the service url in `hero.service.ts` to

```javascript
  private heroesUrl = 'app/heroesx'; 
```

And the service worked again.  That's pretty interesting behavior, but certainly not intuitive... 

The next question was how the name filter worked

```javascript
.get(`app/heroes/?name=${term}`)
```

As I didn't see any functions for that.   Well it turns out this is also a feature of the `InMemoryDataService`.  You can pass a filter for any property of the json, so this will work as well:

```javascript
.get(`app/heroes/?id=${term}`)
```
