---
title: LINQ Cheat Sheet
layout: page
---
# LINQ Cheatsheet

LINQ is a query language in C# and VB.NET designed for querying and updating data.  This data could include an array, object, database, file, etc. 

## References
---

## Select


### Basic Select - C#
```csharp
var query = from m in Manufacturers
	where m.Name == "Trek" 
	select m;

var lambda = Manufacturers.Where(m => m.Name == "Trek").Select(m => m);

```

### Basic Select - VB.NET
```vb
Dim query = from m in Manufacturers
	where m.Name = "Trek" 
	select m

Dim lambda = Manufacturers.Where(function(m) m.Name = "Trek").Select(function(m) m)
```

### Select + new Anonymous Type - C#
```csharp
var query = from m in Manufacturers
	where m.Name == "Trek" 
	select new {Name = m.Name, Key = m.ManufacturerId};

var lambda = Manufacturers.Where(m => m.Name == "Trek").Select(m=> new {Name = m.Name, Key = m.ManufacturerId});
```

### Select + new Anonymous Type - VB.NET
```vb
Dim query = from m in Manufacturers
	where m.Name = "Trek" 
	select new with {.Name = m.Name, .ManufacturerId = m.ManufacturerId}

Dim lambda = Manufacturers.Where(function(m) m.Name = "Trek").Select(function(m) new with {.Name = m.Name, .ManufacturerId = m.ManufacturerId})

```

### Join - C#
```csharp
var query = from m in Models
		join p in PartNumbers on m.ModelId equals p.ModelId
		where m.Name.Contains("Domane")
		select new {m.ModelId, ModelName = m.Name, PartNumber = p.InventoryPartNumber };

var lambda = Models.Join(PartNumbers, m => m.ModelId, p => p.ModelId, (m, p) => new {m = m, p = p} )
					.Where( j => j.m.Name.Contains("Domane"))
					.Select(j => new {ModelId = j.m.ModelId, ModelName = j.m.Name, PartNumber = j.p.InventoryPartNumber});
```

### Join - VB.NET
```vb
Dim query = from m in Models
		join p in PartNumbers on m.ModelId equals p.ModelId
		where m.Name.Contains("Domane")
		select new with {.ModelId = m.ModelId, .ModelName = m.Name, .PartNumber = p.InventoryPartNumber }

Dim lambda = Models.Join(PartNumbers, function(m) m.ModelId, function(p) p.ModelId, function(m,p) new with {.m = m, .p = p} ) _
					.Where( function(j) j.m.Name.Contains("Domane")) _
					.Select(function(j) new with {.ModelId = j.m.ModelId, .ModelName = j.m.Name, .PartNumber = j.p.InventoryPartNumber})
```

### Select Single Record - C#
```csharp
	// Single() will throw an error if more than one result, or there is no result
	var lambda = Statuses.Single( s => s.Name == "Active");

	// SingleorDefault() will throw an error if more than one result, and null if there is no result
	var lambda = Statuses.SingleOrDefault( s => s.Name == "Active");

	// first() will throw an error if no results
	var lambda = PartNumbers.First( p => p.ModelId == 10001);

	//firstOrDefault will return null if no results
	var lambda = PartNumbers.FirstOrDefault( p => p.ModelId == 10001);
	var lambda = PartNumbers.FirstOrDefault();

```

### Select Single Record - VB.Net
```vb
	'Single() will throw an error if more than one result, or there is no result
	Dim lambda = Statuses.Single( function(s) s.Name = "Active")
	
	'SingleorDefault() will throw an error if more than one result, and null if there is no result
	Dim lambda = Statuses.SingleOrDefault( function(s) s.Name = "Active")

	'first() will throw an error if no results
	Dim lambda = PartNumbers.First( function(p) p.ModelId = 10001)
	
	'firstOrDefault will return null if no results
	Dim lambda = PartNumbers.FirstOrDefault( function(p) p.ModelId = 10001)
	
	Dim lambda = PartNumbers.FirstOrDefault()
```