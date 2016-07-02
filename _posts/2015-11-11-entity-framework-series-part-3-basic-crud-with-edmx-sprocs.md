---
id: 405
title: Entity Framework Series Part 3 – Basic CRUD with EDMX + Sprocs
date: 2015-11-11T21:12:48+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=405
permalink: /2015/11/11/entity-framework-series-part-3-basic-crud-with-edmx-sprocs/
categories:
  - Development
  - Projects
tags:
  - Entity Framework
  - SQL Server
  - Visual Studio
---
<span style="font-weight: 400;">In <a href="http://brianvanderplaats.com/2015/10/19/entity-framework-series-part-1-basic-crud-with-edmx/">part I </a>of this series, I created a simple entity data model to demonstrate basic CRUD operations.  I now continue this demonstration using stored procedures exclusively.  Before going into these examples, I’d like to first discuss why one would want to use stored procedures.   </span>

## <span style="font-weight: 400;">Why use Stored Procedures?</span>

<span style="font-weight: 400;">First, I think most developers would agree that a mixture of sprocs and direct table mapping via an ORM is a good approach.  There are simply some types of queries that are more efficient when you craft the SQL directly.  But many queries are quite straightforward, and using an ORM can save a lot of time (both in writing and later deployment).  As I&#8217;ve found out, both EDMX and Code First do a decent job allowing access to sprocs.  However, I’m talking about using sprocs </span>_<span style="font-weight: 400;">exclusively</span>_<span style="font-weight: 400;">.  That is the current technique my company is using, and I suspect a significant number of others companies are using as well.  </span>

Here are a few reasons why I feel this is a valid approach:

<li style="font-weight: 400;">
  <b>Secure the database. </b><span style="font-weight: 400;">  Users may not access tables directly.  Do you want your users writing queries against your OLTP database? How about getting a message from one of your devs, sheepishly telling you that he ran an update against 30K records in the prod database?  Who wants to assign table level security to users?  In my company, users have access to execute sprocs, period.  </span>
</li>
<li style="font-weight: 400;">
  <b>Another layer gives more flexibility in the future </b><span style="font-weight: 400;">&#8211; When tables or relationships change, you can quickly determine exactly how it’s being used by searching for relevant sprocs. You can modify the database directly without having to re-deploy apps, add additional logging or other business logic operations, etc.  </span>
</li>
<li style="font-weight: 400;">
  <b>The database will likely stick around longer than apps</b><span style="font-weight: 400;"> &#8211; This is certainly true for my company.  10 years ago it was web forms and web services, then web forms with business classes, and now looking like MVC/JavaScript + restful API.  Guess what? Still SQL Server!  </span>
</li>
<li style="font-weight: 400;">
  <b>Simpler debugging </b><span style="font-weight: 400;">&#8211; much easier to capture a sproc call, copy to management studio and debug than trap dozens of executing SQL statements.  </span>
</li>
<li style="font-weight: 400;">
  <b>Prevent SQL strings in the UI or business layer.</b><span style="font-weight: 400;">   using Linq  + Entity framework are one thing, but hard coding SQL strings is something no developer should be subject to&#8230;</span>
</li>
<li style="font-weight: 400;">
  <b>Use of SSRS / Crystal reports</b><span style="font-weight: 400;"> &#8211; need to use Sprocs (no LINQ, at least not without some </span><a href="http://codebetter.com/petervanooijen/2009/07/01/reporting-against-a-domain-model/"><span style="font-weight: 400;">workarounds</span></a><span style="font-weight: 400;">)</span>
</li>
<li style="font-weight: 400;">
  <b>Past experience supporting legacy apps &#8211; </b><span style="font-weight: 400;">with haphazard SQL calls, in event handlers, classes, init code, etc.  </span>
</li>
<li style="font-weight: 400;">
  <b>Large data model used by diverse set of apps</b><span style="font-weight: 400;"> &#8211; 250,500, or 1000K+ entities.  </span>
</li>
<li style="font-weight: 400;">
  <b>Existing Sprocs</b><span style="font-weight: 400;"> &#8211; You have hundreds or thousands of existing sprocs in place already.  </span>
</li>

<span style="font-weight: 400;">I do not however subscribe to the following arguments:</span>

<li style="font-weight: 400;">
  <b>Prevent developers from writing SQL / knowledge of table structures</b><span style="font-weight: 400;"> &#8211;  In my company, developers write the sprocs, the dba (me) reviews, then commits.  </span>
</li>
<li style="font-weight: 400;">
  <b>Entity Framework / ORM’s generate inefficient SQL </b><span style="font-weight: 400;">&#8211; I don’t find this to be the case, generally.  In my earlier examples, EF wrote practically the same SQL as I did.  Regardless of whether you use an ORM or sprocs, you should have profiler running to see what your code is actually doing to the database.  </span>
</li>

<span style="font-weight: 400;">Now, I’d be happy to discuss the merits of these with any developer and I’m not claiming that direct table access is bad universally. My chief concern is this:  If a data access technique (namely data access through sprocs)  is widely accepted and/or has a large legacy codebase, then it should be supported by Microsoft’s current flavor of data access tech.  I’m actually considering going the direct-table approach for some future apps, and knowing that I can still make use of existing sproc infrastructure is crucial.  </span>

## <span style="font-weight: 400;">Entity Framework and Sprocs</span>

<span style="font-weight: 400;">So how well does Entity Framework support sprocs? “it depends”.   In my research, EF 6 / EDMX has good support for sprocs, and Code First sprocs sort of work (which I will go into in a future post).    </span>

<span style="font-weight: 400;">When thinking about how an ORM should interact with sprocs, we want to see the following happen, and EF more or less supports these:  </span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">It should work with complex types</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">It should handle parameters well, including output params.  </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">It should detect sproc signatures and return types automatically so we don’t need to hand-write properties, getters and setters, etc.  </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">It should allow us to write similar code for table-direct or sproc calls</span>
</li>

<span style="font-weight: 400;">The rest of this post details how this works with EDMX.  As usual, the source is on </span>[<span style="font-weight: 400;">github </span>](https://github.com/brianvp/entityframework-examples/tree/master/EDMXBasicCRUDSprocs)

## <span style="font-weight: 400;">Function Imports</span>

<span style="font-weight: 400;">The primary way edmx interacts with sprocs is through “function imports”, and it is very easy to add these to a new or existing data model.  When in the wizard, you simply choose the sprocs you need (and you can easily add more in the future by running the Update Data Model from Database wizard):</span>

[<img class="alignnone size-full wp-image-406" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/EntityDataModelAddSprocs.png" alt="EntityDataModelAddSprocs" width="621" height="559" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/EntityDataModelAddSprocs.png)

<span style="font-weight: 400;">For each sproc selected, a function import reference is added in the data model.  You can access this function by simply calling</span>

<pre class="brush: csharp; title: ; notranslate" title="">db.ModelSelectByKey(modelId)

</pre>

<span style="font-weight: 400;">If the sproc returns a resultset, a new POCO class is added to the model as well, with a name of <sproc name>_result</span>

<pre class="brush: csharp; title: ; notranslate" title="">public partial class ModelSelectByKey_Result
{
 public int ModelId { get; set; }
 public string Name { get; set; }
 public string ManufacturerCode { get; set; }
...
}

</pre>

<span style="font-weight: 400;">Putting this together in our sample program, here is a simple sproc call:</span>

<pre class="brush: csharp; title: ; notranslate" title="">using (var db = new BikeStoreEntities())
 {

 var models = db.ModelSelectFilter(null,null,null,null,null,null,null,null,null);

 foreach (var model in models)
 {
 Console.WriteLine(model.ModelId + " | " + model.Name);
 }
}
</pre>

you can view the current list of function imports through the model browser:
  
[<img class="alignnone size-full wp-image-416" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/EdmxFunctionImports.png" alt="EdmxFunctionImports" width="351" height="507" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/EdmxFunctionImports.png)

## <span style="font-weight: 400;">Stored Procedure Mapping</span>

<span style="font-weight: 400;">While calling functions works well, this doesn’t make use of the primary advantage of EF &#8211; working with entities.  We want to load a model, update it, create a new model, delete, from a “Model” class, not just call a bunch of functions with parameters.  This is accomplished through “CUD” mapping.  You cannot map the “Read” operation, which is odd because as you’ll see in a moment, it is trivial to populate a list of Model entities directly from a sproc&#8230;  For the other three operations, you simply select the entity in the model diagram, and choose the “stored procedure mapping&#8221;.  From there you can provide an Insert, Delete, and Update sproc.  </span>

<span style="font-weight: 400;">Here’s what each looks like for the Model entity</span>

[<img class="alignnone size-full wp-image-407" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/ModelInsertMapping.png" alt="ModelInsertMapping" width="794" height="326" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/ModelInsertMapping.png)

Note you must provide a rows affected output for an identity column (ModelId) if you want to access this after insert.

[<img class="alignnone size-full wp-image-408" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/ModelUpdateMapping.png" alt="ModelUpdateMapping" width="764" height="301" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/ModelUpdateMapping.png)

[<img class="alignnone size-full wp-image-409" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/ModelDeleteMapping.png" alt="ModelDeleteMapping" width="763" height="67" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/ModelDeleteMapping.png)

<span style="font-weight: 400;">The sprocs will be utilized when you invoke the SaveChanges() method of the context.  Note that mapping is an all or nothing approach.  You can’t use an insert sproc and then direct table access for updates &#8211; you need to provide both an insert and update sproc.  However, you do not need to provide the sproc if your code avoids that particular option i.e. you can omit the delete sproc if you never call delete on the model.  </span>

### <span style="font-weight: 400;">Create</span>

<span style="font-weight: 400;">Below is the code for creating a new Model, and it looks no different from the standard direct table access of EF:</span>

<pre class="brush: csharp; title: ; notranslate" title="">int modelId = 0;

 using (var db = new BikeStoreEntities())
 {
 var model = new Model { Name = "Domane 5.2", ListPrice = 3499.99m };

db.Models.Add(model);
db.SaveChanges();

modelId = model.ModelId;

}

Console.WriteLine("New Model: " + modelId);

</pre>

<span style="font-weight: 400;">Notice that ModelInsert has an output parameter, which isn’t supported by EF very well.  While I was able to get this sproc working while retaining the output parameter, I wasn’t able to determine how to extract the result after SaveChanges(), so I decided to simply change the ModelInsert sproc to follow the pattern of selecting the ID after insert (which is how EF would generate the T-SQL).   The happy result is that EF is able to map this to the ModelId property, so as long as you are willing to make that change, this is good.  </span>

### <span style="font-weight: 400;">Update</span>

<pre class="brush: csharp; title: ; notranslate" title="">using (var db = new BikeStoreEntities())
 {

 var model = db.ModelSelectByKey(modelId).FirstOrDefault();

 if (model != null)
 {

 Model m = new Model();

 m.ModelId = model.ModelId;
 m.Name = model.Name;
 m.ManufacturerCode = model.ManufacturerCode;
 m.CategoryId = model.CategoryId;
 m.Description = model.Description;
 m.Features = model.Features;
 m.StatusId = model.StatusId;
 m.ManufacturerId = model.ManufacturerId;
 m.ListPrice = model.ListPrice;
 m.ImageCollection = model.ImageCollection;
 m.CategoryCustomData = model.CategoryCustomData;
 m.ManufacturerCustomData = model.ManufacturerCustomData;

 db.Models.Attach(m);

 // make the change - needs to happen after the attach, otherwise the change
 // will not be registered
 m.Features = "500 Series OCLV Frame";

 db.SaveChanges();
 } 
 }
</pre>

<span style="font-weight: 400;">Above was my first attempt at writing an update sequence, and you can see this is not an efficient solution.   Who wants to write out all those extra mapping lines?  Not to mention when fields change…  The solution is to change the ModelSelectByKey Function Import from using a complex return type of ModelSelectByKey_Result to a collection of Model entities</span>

[<img class="alignnone size-full wp-image-410" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/EditFunctionImport.png" alt="EditFunctionImport" width="499" height="613" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/EditFunctionImport.png)

&nbsp;

<span style="font-weight: 400;">Now we can simplify the code to this:</span>

<pre class="brush: csharp; title: ; notranslate" title="">using (var db = new BikeStoreEntities())
 {

 var model = db.ModelSelectByKey(modelId).FirstOrDefault();

 if (model != null)
 {
 model.Features = "500 Series OCLV Frame";

 db.SaveChanges();
 } 
 }
</pre>

<span style="font-weight: 400;">The only problem with this is if your sproc contains additional fields not part of the Model entity, they won’t be accessible.  </span>

### <span style="font-weight: 400;">Delete</span>

<pre class="brush: csharp; title: ; notranslate" title="">using (var db = new BikeStoreEntities())
 {
 // uses sprocs when mapping set in model
 var model = new Model { ModelId = modelId };

 db.Models.Attach(model);
 db.Models.Remove(model);
 db.SaveChanges();
 }
</pre>

<span style="font-weight: 400;">Not much to see here, works as expected.  At this point we’ve replicated all of the key CRUD operations in the earlier examples. Below is a trace of the entire application running in SQLProfiler:</span>

[<img class="alignnone size-full wp-image-411" src="http://brianvanderplaats.com/wp-content/uploads/2015/11/EDMXSprocProfilerTrace.png" alt="EDMXSprocProfilerTrace" width="660" height="298" />](http://brianvanderplaats.com/wp-content/uploads/2015/11/EDMXSprocProfilerTrace.png)

## <span style="font-weight: 400;">Direct Procedure Calls</span>

<span style="font-weight: 400;">Before wrapping up, let’s talk about direct sproc calls.  In the example below, instead of using a function import, we can call the SqlQuery() method and pass the name of the sproc as well as any necessary parameters:</span>

<pre class="brush: csharp; title: ; notranslate" title="">using (var db = new BikeStoreEntities())
 {

 var name = new SqlParameter("@name", DBNull.Value);
 var manufacturerCode = new SqlParameter("@manufacturercode", DBNull.Value);
 var categoryName = new SqlParameter("@categoryname", DBNull.Value);
 var description = new SqlParameter("@description", DBNull.Value);
 var features = new SqlParameter("@features", DBNull.Value);
 var minListPrice = new SqlParameter("@minListPrice", DBNull.Value);
 var maxListPrice = new SqlParameter("@maxListPrice", DBNull.Value);
 var statusName = new SqlParameter("@statusName", DBNull.Value);
 var manufacturerName = new SqlParameter("@manufacturerName", DBNull.Value);

 var models = db.Database.SqlQuery<ModelSelectFilter_Result>("product.ModelSelectFilter @name, @manufacturercode, @categoryname,@description,@features,@minListPrice,@maxListPrice,@statusName, @manufacturerName", 
 name,manufacturerCode, categoryName, description, features, minListPrice,maxListPrice,statusName,manufacturerName).ToList();

 foreach (var model in models)
 {
 Console.WriteLine(model.ModelId + " | " + model.Name);
 }

 }
</pre>

<span style="font-weight: 400;">Not a very efficient technique, especially when using a number of parameters.  At this point you are simply using EF as a wrapper for ADO.NET, so if this is your main technique, why bother using EF at all?</span>

## <span style="font-weight: 400;">Conclusion</span>

<span style="font-weight: 400;">All in all, sproc access in EDMX is not that bad, and lets us work with entities to a large degree.  If your apps are stored procedure-centric, there is a lot of built-in support in Entity Framework that you should find useful.</span>