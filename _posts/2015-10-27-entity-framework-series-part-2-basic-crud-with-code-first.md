---
id: 389
title: 'Entity Framework Series Part 2 &#8211; Basic CRUD with Code First'
date: 2015-10-27T21:47:37+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=389
permalink: /2015/10/27/entity-framework-series-part-2-basic-crud-with-code-first/
categories:
  - Development
  - Projects
tags:
  - Entity Framework
  - SQL Server
  - Visual Studio
---
The second part of my Entity Framework series is identical to the [first](http://brianvanderplaats.com/2015/10/19/entity-framework-series-part-1-basic-crud-with-edmx/), except this time I&#8217;m using Code First.  I&#8217;m not going to go into as much detail, but will show the differences.  As usual the code is hosted on [github](https://github.com/brianvp/entityframework-examples/tree/master/CodeFirstBasicCRUD).

## <span style="font-weight: 400;">Add the DbContext & Generate POCO Classes</span>

With Code First, there is no .edmx file, just a DbContext class and a set of POCO classes.  You will use the DbContext to load the POCO classes from the database.  Note that you should name your model <descriptor>Context vs. <descriptor>Model.   This is different than edmx, and confusing.

[<img class="alignnone size-full wp-image-391" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/AddNewEntityContext.png" alt="AddNewEntityContext" width="948" height="574" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/AddNewEntityContext.png)

You&#8217;ll then select Code First from database:

[<img class="alignnone size-full wp-image-390" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardDesignerCodeFirst.png" alt="EntityDataModelWizardDesignerCodeFirst" width="623" height="556" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardDesignerCodeFirst.png)

And choose your objects: (note that Stored Procedures and Functions are not available options)

[<img class="alignnone size-full wp-image-392" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardCodeFirstChooseDatabaseObjects.png" alt="EntityDataModelWizardCodeFirstChooseDatabaseObjects" width="624" height="560" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/EntityDataModelWizardCodeFirstChooseDatabaseObjects.png)

After you finish the wizard, your project tree will look as follows: (you should probably put these into a sub folder e.g. \models)

[<img class="alignnone size-full wp-image-393" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreCodeFirstClasses.png" alt="BikeStoreCodeFirstClasses" width="270" height="235" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreCodeFirstClasses.png)

## <span style="font-weight: 400;">Accessing the Context</span>

From here the code is almost identical to the edmx version, with a few key differences.  First, instead of:

```csharp
using (var db = new BikeStoreEntities())
 {
 //do database stuff
 }

```

You  call this:

```csharp
using (var db = new BikeStoreContext())
 {
 //do database stuff
 }

```

Second, while the SQL executed is virtually identical, the first time the context is accessed, the following SQL executes:

```sql
IF db_id(N'BikeStore') IS NOT NULL SELECT 1 ELSE SELECT Count(*) FROM sys.databases WHERE [name]=N'BikeStore'

SELECT Count(*)
FROM INFORMATION_SCHEMA.TABLES AS t
WHERE t.TABLE_SCHEMA + '.' + t.TABLE_NAME IN ('product.Category','product.Model','product.Manufacturer','product.PartNumber','product.Status')
 OR t.TABLE_NAME = 'EdmMetadata'
 
exec sp_executesql N'SELECT 
 [GroupBy1].[A1] AS [C1]
 FROM ( SELECT 
 COUNT(1) AS [A1]
 FROM [dbo].[__MigrationHistory] AS [Extent1]
 WHERE [Extent1].[ContextKey] = @p__linq__0
 ) AS [GroupBy1]',N'@p__linq__0 nvarchar(4000)',@p__linq__0=N'CodeFirstBasicCRUD.BikeStoreContext'
SELECT
[GroupBy1].[A1] AS [C1]
FROM ( SELECT
COUNT(1) AS [A1]
FROM [dbo].[__MigrationHistory] AS [Extent1]
) AS [GroupBy1]

SELECT TOP (1)
[Extent1].[Id] AS [Id],
[Extent1].[ModelHash] AS [ModelHash]
FROM [dbo].[EdmMetadata] AS [Extent1]
ORDER BY [Extent1].[Id] DESC
```

These SQL calls were not present in any of the edmx traces &#8211; these are support for Code First [Migrations](https://msdn.microsoft.com/en-us/data/jj591621.aspx).  We can turn this off in the constructor of our context class:

```csharp
public BikeStoreContext()
            : base("name=BikeStoreContext")
        {
            Database.SetInitializer<BikeStoreContext>(null); 
        }
```

## Conclusion

At this point we have two approaches that achieve the same basic goal, but the basic question is why are there two approaches?  Apparently Microsoft feels this is a bad idea, so in EF 7 the emdx approach is being [dropped](http://blogs.msdn.com/b/adonet/archive/2014/10/21/ef7-what-does-code-first-only-really-mean.aspx).  Although it&#8217;s still early in my investigation, Code First feels cleaner, and I certainly have no love for large xml-based designer files.

&nbsp;

&nbsp;

&nbsp;