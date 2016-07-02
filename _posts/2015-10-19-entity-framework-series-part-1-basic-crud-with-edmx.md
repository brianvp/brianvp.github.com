---
id: 373
title: 'Entity Framework Series Part 1 &#8211; Basic CRUD with EDMX'
date: 2015-10-19T21:09:52+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=373
permalink: /2015/10/19/entity-framework-series-part-1-basic-crud-with-edmx/
categories:
  - Development
  - Projects
tags:
  - 'c#'
  - Entity Framework
  - SQL Server
  - Visual Studio
---
<span style="font-weight: 400;">As a first of a multi-part series, I am exploring <a href="https://msdn.microsoft.com/en-us/data/ee712907.aspx">Entity Framework 6.X</a>.  My goal is to begin with simple examples, working up to using Entity Framework in a production app.  I will explore both the designer/EDMX style, as well as CodeFirst, using Visual Studio 2015.  In all cases, I will be focusing on an </span>_<span style="font-weight: 400;">existing </span>_<span style="font-weight: 400;">database &#8211; the </span>[<span style="font-weight: 400;">Bike Store</span>](http://brianvanderplaats.com/2015/10/12/creating-the-bike-store-database-part-i-product-schema/) <span style="font-weight: 400;">database.   Apparently the next version of Entity Framework, EF 7, will <a href="http://blogs.msdn.com/b/adonet/archive/2014/05/19/ef7-new-platforms-new-data-stores.aspx">not include</a> the EDMX/designer option.  Because of this, I seriously considered ignoring EDMX all together and focusing on CodeFirst, however A) Microsoft will support EF 6 for many years yet and B) I want to understand the problems EDMX is trying to solve, and see if CodeFirst is a sufficient replacement.  </span>

<span style="font-weight: 400;">For this first post, we will look at simple </span>[<span style="font-weight: 400;">CRUD</span>](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) <span style="font-weight: 400;">operations using an EDMX model.  You can see the full code on <a href="https://github.com/brianvp/entityframework-examples/tree/master/EDMXBasicCRUD">github</a>. </span>

## <span style="font-weight: 400;">Adding the Model</span>

<span style="font-weight: 400;">The simplest approach to demonstrate EDMX is using a console application.  After creating the project, you&#8217;ll need to add a new ADO.Net Entity Data Model:</span>

[<img class="alignnone size-full wp-image-374" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/AddNewEntityDataModel.png" alt="AddNewEntityDataModel" width="938" height="657" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/AddNewEntityDataModel.png)

<span style="font-weight: 400;">Choose the EF Designer From database option:</span>

[<img class="alignnone size-full wp-image-375" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardEFDesignerFromDatabase.png" alt="EntityDataModelWizardEFDesignerFromDatabase" width="629" height="562" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardEFDesignerFromDatabase.png)

<span style="font-weight: 400;">Then choose which tables to bring over (you can add views and sprocs as well &#8211; I will cover this in a future post).  </span>

[<img class="alignnone size-full wp-image-376" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardChooseDatabaseObjects.png" alt="EntityDataModelWizardChooseDatabaseObjects" width="626" height="560" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardChooseDatabaseObjects.png)

<span style="font-weight: 400;">Select version 6.X:</span>

[<img class="alignnone size-full wp-image-377" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardChooseEFVersion.png" alt="EntityDataModelWizardChooseEFVersion" width="621" height="552" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardChooseEFVersion.png)

<span style="font-weight: 400;">With the wizard completed, your project will contain a new .edmx file and a sub-tree containing the POCO classes, context class, templates, and designer files:</span>

[<img class="alignnone size-full wp-image-378" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreModelEDMXFileLayout.png" alt="BikeStoreModelEDMXFileLayout" width="293" height="236" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreModelEDMXFileLayout.png)

<span style="font-weight: 400;">Opening the edmx designer, the tables look as follows (after manual positioning):</span>

[<img class="alignnone size-full wp-image-379" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreModelEDMXLayout.png" alt="BikeStoreModelEDMXLayout" width="463" height="868" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreModelEDMXLayout.png)

<span style="font-weight: 400;">In solution explorer, if you select the edmx file, a new tab appears that displays a sub-tree of model contents (note this is where sprocs can be added to the model)</span>

[<img class="alignnone size-full wp-image-380" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreModelModelBrowser.png" alt="BikeStoreModelModelBrowser" width="318" height="381" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreModelModelBrowser.png)

## <span style="font-weight: 400;">Accessing the Model</span>

<span style="font-weight: 400;">The first step to using the new entities is to create a database context object.  I recommend sticking to the standard practice of wrapping the call in a using block:</span>

```csharp
using (var db = new BikeStoreEntities())
 {
 //do database stuff
 }

```

<span style="font-weight: 400;">The rest of this post will demonstrate six common data usage patterns:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Selecting an entire table</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Executing a filtered query</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Creating a new record</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Selecting a single record</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Updating a record</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Deleting a record</span>
</li>

<span style="font-weight: 400;">In addition to the c# code, I will post the generated </span>[<span style="font-weight: 400;">SQL Profiler</span>](https://msdn.microsoft.com/en-us/library/ms181091.aspx) <span style="font-weight: 400;">trace.  (I will write a future post on SQL profiler, but suffice it to say, if you do any serious data work, you need to be using this tool!)</span>

### <span style="font-weight: 400;">Select Table</span>

```csharp
using (var db = new BikeStoreEntities())
 {
 var query = from m in db.Models
 orderby m.Name
 select m;

Console.WriteLine("All Models in the Database");

foreach (var model in query)
 {
 Console.WriteLine(model.ModelId + " | " + model.Name);
 }
 }

```

```sql
SELECT 
 [Extent1].[ModelId] AS [ModelId], 
 [Extent1].[Name] AS [Name], 
 [Extent1].[ManufacturerCode] AS [ManufacturerCode], 
 [Extent1].[CategoryId] AS [CategoryId], 
 [Extent1].[Description] AS [Description], 
 [Extent1].[Features] AS [Features], 
 [Extent1].[StatusId] AS [StatusId], 
 [Extent1].[ManufacturerId] AS [ManufacturerId], 
 [Extent1].[ListPrice] AS [ListPrice], 
 [Extent1].[ImageCollection] AS [ImageCollection], 
 [Extent1].[CategoryCustomData] AS [CategoryCustomData], 
 [Extent1].[ManufacturerCustomData] AS [ManufacturerCustomData], 
 [Extent1].[DateModified] AS [DateModified], 
 [Extent1].[DateCreated] AS [DateCreated]
 FROM [product].[Model] AS [Extent1]
 ORDER BY [Extent1].[Name] ASC
```

Note that the SQL above is not executed until the query is accessed in the foreach block. This is known as [deferred execution](https://msdn.microsoft.com/en-us/library/vstudio/bb738633(v=vs.100).aspx).  After execution, the result set is kept in memory for further iterations through the loop.

### <span style="font-weight: 400;">Execute a Filtered Query</span>

A common SQL pattern is joining multiple tables into a custom result set.  Native SQL Code is tough to beat here, but we&#8217;ll see how Entity Framework does:

```csharp
using (var db = new BikeStoreEntities())
 {
 var query = from md in db.Models
 join pn in db.PartNumbers
 on md.ModelId equals pn.ModelId
 join ct in db.Categories
 on md.CategoryId equals ct.CategoryId
 where md.Name.Contains("tape")
 select new { md.ModelId, ModelName = md.Name, PartNumberName = pn.Name, pn.InventoryPartNumber, pn.ListPrice, CategoryName = ct.Name };

 Console.WriteLine("All Bar Tape Products in the Database");

 foreach (var item in query)
 {
 Console.WriteLine(item.ModelId + " | " + item.ModelName + " | " + item.PartNumberName + " | " + item.InventoryPartNumber + " | " + item.ListPrice + " | " + item.CategoryName);
 }
 }
```

```sql
SELECT 
 [Extent1].[ModelId] AS [ModelId], 
 [Extent1].[Name] AS [Name], 
 [Extent2].[Name] AS [Name1], 
 [Extent2].[InventoryPartNumber] AS [InventoryPartNumber], 
 [Extent2].[ListPrice] AS [ListPrice], 
 [Extent3].[Name] AS [Name2]
 FROM [product].[Model] AS [Extent1]
 INNER JOIN [product].[PartNumber] AS [Extent2] ON [Extent1].[ModelId] = [Extent2].[ModelId]
 INNER JOIN [product].[Category] AS [Extent3] ON [Extent1].[CategoryId] = [Extent3].[CategoryId]
 WHERE [Extent1].[Name] LIKE N'%tape%'
```

Not too bad &#8211; the linq code is very similar to the generated SQL Code.

### <span style="font-weight: 400;">Create a New Record</span>

```sql
using (var db = new BikeStoreEntities())
 {
 var model = new Model { Name = "Domane 5.2", ListPrice = 3499.99m };

 db.Models.Add(model);
 db.SaveChanges();

 modelId = model.ModelId;
 }
```

```sql
exec sp_executesql N'INSERT [product].[Model]([Name], [ManufacturerCode], [CategoryId], [Description], [Features], [StatusId], [ManufacturerId], [ListPrice], [ImageCollection], [CategoryCustomData], [ManufacturerCustomData], [DateModified], [DateCreated])
VALUES (@0, NULL, NULL, NULL, NULL, NULL, NULL, @1, NULL, NULL, NULL, NULL, NULL)
SELECT [ModelId]
FROM [product].[Model]
WHERE @@ROWCOUNT &gt; 0 AND [ModelId] = scope_identity()',N'@0 nvarchar(100),@1 decimal(19,4)',@0=N'Domane 5.2',@1=3499.9900
```

Notice the select immediately following the insert operation. Scope_Identity() retrieves the identity value of ModelId for the inserted row. Entity Framework then maps this into the ModelId property of the model reference we added.

### <span style="font-weight: 400;">Select a Single Record</span>

```csharp
using (var db = new BikeStoreEntities())
 {
 var model = db.Models.SingleOrDefault(b =&gt; b.ModelId == modelId);

 Console.WriteLine("Selected Model:");

 if ( model != null)
 {
 Console.WriteLine(model.ModelId + " | " + model.Name);
 }
 }
```

```sql
exec sp_executesql N'SELECT TOP (2) 
 [Extent1].[ModelId] AS [ModelId], 
 [Extent1].[Name] AS [Name], 
 [Extent1].[ManufacturerCode] AS [ManufacturerCode], 
 [Extent1].[CategoryId] AS [CategoryId], 
 [Extent1].[Description] AS [Description], 
 [Extent1].[Features] AS [Features], 
 [Extent1].[StatusId] AS [StatusId], 
 [Extent1].[ManufacturerId] AS [ManufacturerId], 
 [Extent1].[ListPrice] AS [ListPrice], 
 [Extent1].[ImageCollection] AS [ImageCollection], 
 [Extent1].[CategoryCustomData] AS [CategoryCustomData], 
 [Extent1].[ManufacturerCustomData] AS [ManufacturerCustomData], 
 [Extent1].[DateModified] AS [DateModified], 
 [Extent1].[DateCreated] AS [DateCreated]
 FROM [product].[Model] AS [Extent1]
 WHERE [Extent1].[ModelId] = @p__linq__0',N'@p__linq__0 int',@p__linq__0=11043
```

### <span style="font-weight: 400;">Update a Record</span>

```csharp
using (var db = new BikeStoreEntities())
 {
 var model = db.Models.SingleOrDefault(b =&gt;; b.ModelId == modelId);

 if (model != null)
 {
 model.Features = "500 Series OCLV Frame";

 db.SaveChanges();
 }
 }
```

(excluding SQL generated by SingleOrDefault()

```sql
exec sp_executesql N'UPDATE [product].[Model]
SET [Features] = @0
WHERE ([ModelId] = @1)
',N'@0 nvarchar(max) ,@1 int',@0=N'500 Series OCLV Frame',@1=11043
```

This is quite interesting &#8211; the update statement generates a custom list of update fields based on the state of the model. Definitely a situation where auto generation of SQL is useful.  Also note that no update statement(s) are sent to SQL Server until db.SaveChanges() is called.

### <span style="font-weight: 400;">Delete a Record</span>

```csharp
using (var db = new BikeStoreEntities())
 {
 var model = new Model { ModelId = modelId };

 db.Models.Attach(model);
 db.Models.Remove(model);
 db.SaveChanges();
 }
```

Many examples have you populate an entity via single select before deleting, but the above approach does generate a select statement before deleting.

```sql
exec sp_executesql N'DELETE [product].[Model]
WHERE ([ModelId] = @0)',N'@0 int',@0=11043
```

## Conclusion

These examples demonstrate basic access to a SQL Database through Entity Framework.  So far I am impressed with the SQL generation capabilities.  The generated SQL is basically the same as I would write if I were writing the SQL manually.  A key benefit to writing SQL is using SSMS and testing queries immediately.  While this isn't as easy in Visual Studio, I recommend [LINQPad ](http://www.linqpad.net/)for this purpose.  It even pluralizes the entity / table names for you! (category table in database referenced as categories in LINQ)