---
title: 'Default Startup Project in Visual Studio'
date: 2016-12-07T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/12/07/Default-Startup-Project-in-Visual-Studio/
categories:
  - General
tags:
  - Visual Studio
---
Had an interesting error the other day.  While reviewing an interns test project, I noticed that on a fresh clone of his repo, his website errored out on startup.  For our projects we typically have a project for the website, and another for the business classes.  For some reason, Visual Studio was trying to run the business project as the website.  

This is easily fixed right-clicking the web project and selecting `set as startup project`, but this setting is stored in the `.suo` file which is *not* checked in.  So why does this work in our other projects?  

Doing some googling, came across this [stackoverflow answer](http://stackoverflow.com/a/1808352/24892).  It turns out that Visual Studio looks at the order of the projects in the solution file.  Somehow, my intern added the business project first - I don't think anyone has done that before.   

All you need to do is manually edit the `.sln` file in a text editor, and move the "startup" project to the top of t he `Project` / `EndProject` blocks

```
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio 14
VisualStudioVersion = 14.0.23107.0
MinimumVisualStudioVersion = 10.0.40219.1
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "MySiteApp", "MySite.App\MySiteApp.csproj", "{F5C8B11C-FF25-45B6-8F25-B1C779D4A053}"
EndProject
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "MySiteBusiness", "MySite.Business\MySiteBusiness.csproj", "{E9561B89-36A0-43C2-B9BA-970843316D23}"
EndProject
```

On a related note, it seems like Microsoft can't pick a file format and stay with it.  the `.suo` file is binary, `.config` files are xml, and the `.sln` file is basically just simple keys and values.   
