---
id: 332
title: 'SQL Server Tip #1: Basic Table Template'
date: 2015-10-06T21:45:31+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=332
permalink: /2015/10/06/sql-server-tip-1-basic-table-template/
categories:
  - SQL-Server-Tips
tags:
  - SQL Server
---
A common SQL task is to set up new tables for an application, and while you can use the GUI for this, I prefer using scripts. It’s faster, and leads to better deployment practices. (The one time I do find the designer useful is when making a column size change to a table.  In this case, I will make the change in the designer, and generate a script.  It looks ugly, but saves a lot of manual coding)

[<img class="alignnone size-medium wp-image-334" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/SqlTip1_TableDesigner-241x300.png" alt="SqlTip1_TableDesigner" width="241" height="300" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/SqlTip1_TableDesigner.png)[<img class="alignnone size-medium wp-image-335" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/SqlTip1_TableDesignerChangeScript-300x238.png" alt="SqlTip1_TableDesignerChangeScript" width="300" height="238" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/SqlTip1_TableDesignerChangeScript.png)

Since I don’t typically create tables every day, even as a dba, I like using a basic template. You can script out a template using an existing SQL Server table, which looks like this:

<pre class="brush: sql; title: ; notranslate" title="">/****** Object: Table [product].[Category] Script Date: 10/6/2015 8:38:18 PM ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [product].[Category](
[CategoryId] [int] IDENTITY(100,1) NOT NULL,
[Name] [nvarchar](50) NULL,
[CustomDataFields] [nvarchar](max) NULL,
[DateModified] [datetime2](7) NULL,
[DateCreated] [datetime2](7) NULL DEFAULT (getdate()),
PRIMARY KEY CLUSTERED
(
[CategoryId] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]

GO
</pre>

Or if you are using SQL 2012 or later you can use code snippets:

<pre class="brush: sql; title: ; notranslate" title="">CREATE TABLE dbo.Sample_Table
(
column_1 int NOT NULL,
column_2 int NULL
);
</pre>

I don’t care for these much (too verbose or too simplistic), so I created my own:

<pre class="brush: sql; title: ; notranslate" title="">CREATE TABLE tempdb.dbo.temp1
(
 Temp1Id INT IDENTITY(1,1) NOT NULL
 CONSTRAINT temp1PrimaryKey PRIMARY KEY CLUSTERED,
 VALUE VARCHAR(25),
 DateModified datetime2,
 DateCreated datetime2 default getdate()
)
GO

CREATE TABLE tempdb.dbo.temp2
(
 Temp2Id INT IDENTITY(1,1) NOT NULL
 CONSTRAINT temp2PrimaryKey PRIMARY KEY CLUSTERED,
 Temp1Id INT NOT NULL
 CONSTRAINT Temp2Temp1ForeignKey FOREIGN KEY (Temp1Id)
 REFERENCES dbo.temp1(Temp1Id),
 VALUE VARCHAR(25) NULL,
 DateModified datetime2,
 DateCreated datetime2 default getdate()
)
GO
</pre>

[Gist](https://gist.github.com/brianvp/8bfb8aa3d9ec054e5531)

At my current job, I allow developers to create their own tables (following standards of course!), and a very common code review item is that they forget to make the foreign key constraint! It’s simpler to add this at the time of creation. Sometimes this makes data insertion a problem when people are “testing” &#8211; but it&#8217;s far worse to have this go uncaught when deploying to production.

Additionally, unless you are using database projects, it’s a good idea to keep all your table creation logic in a single script. This will save a lot of time when deploying to staging or production.