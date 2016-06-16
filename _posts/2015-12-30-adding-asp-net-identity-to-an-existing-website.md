---
id: 450
title: Adding ASP.NET Identity to an Existing Website
date: 2015-12-30T20:40:00+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=450
permalink: /2015/12/30/adding-asp-net-identity-to-an-existing-website/
categories:
  - Development
  - General
tags:
  - ASP.NET Identity
  - Entity Framework
  - NuGet
---
Usually when working with ASP.Net Identity all the necessary components are added with the existing project scaffolding, but recently I needed to add an Identity database to an existing project. Below is simple way to accomplish this.

### Step 1 &#8211; Install NuGet packages

From the Package Manager Console run:

<pre class="brush: csharp; title: ; notranslate" title="">Install-Package Microsoft.AspNet.Identity.EntityFramework
</pre>

(this will install AspNet.Identity.Core as well)

### Step 2 &#8211; Create Context Class

Create a new empty Entity Framework Code First context.   Replace the default class definition with:

<pre class="brush: csharp; title: ; notranslate" title="">...
 using Microsoft.AspNet.Identity;
 using Microsoft.AspNet.Identity.EntityFramework;

 public class SecurityDbContext : IdentityDbContext&lt;IdentityUser&gt;
 {
...
</pre>

Update your web.config file to point the new context to a valid SQL Database.

note &#8211; If your context class overrides OnModelCreating, make sure base.OnModelCreating() is called

### Step 3 &#8211; Enable Migrations & Populate Seed Method

In Package Manager Console run:

<pre class="brush: csharp; title: ; notranslate" title="">Enable-Migrations -EnableAutomaticMigrations

</pre>

Then, optionally, you can create additional users or roles as needed in \Migrations\Configurations.cs:

<pre class="brush: csharp; title: ; notranslate" title="">protected override void Seed(MyApplication.Models.SecurityDbContext context)
 {
 var um = new UserManager&lt;IdentityUser&gt;(new UserStore&lt;IdentityUser&gt;(context));

 var user = new IdentityUser()
 {
 UserName = "admin"
 };

 IdentityResult ir = um.Create(user, "1234#abcD");

 }

</pre>

### Step 4 &#8211; Update Database

In Package Manger Console run

<pre class="brush: csharp; title: ; notranslate" title="">Update-Database
</pre>

The Identity tables should now exist in the database:

[<img class="alignnone size-full wp-image-451" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/IdentityTables.png" alt="IdentityTables" width="233" height="162" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/IdentityTables.png)

&nbsp;

&nbsp;