---
id: 360
title: 'Creating the Bike Store Database &#8211; Part I &#8211; Product Schema'
date: 2015-10-12T22:03:44+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=360
permalink: /2015/10/12/creating-the-bike-store-database-part-i-product-schema/
categories:
  - Projects
tags:
  - Database
  - GitHub
  - SQL Server
  - Visual Studio
---
Over the next few months I plan on doing a bunch of research on Entity Framework, WebAPI, MVC, and AngularJS.   My focus will be on how these technologies can be used to build enterprise level applications.   To accomplish this I need a sample database that properly models typical enterprise entities.   To this end, I’ve created a database for a fictional “Bike Store”.   At the bike store, we will need to track a product catalog, inventory, customers, sales, employees, and so on.  Below is a  high level overview of the data needs: 

[<img class="alignnone size-full wp-image-361" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/BikesStoreDatabaseArchitecture.png" alt="BikesStoreDatabaseArchitecture" width="562" height="372" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/BikesStoreDatabaseArchitecture.png)

At this time I’m only implementing the Product area of the database schema, but you can see how it fits into the larger theoretical schema.  This represents a fairly standard view of any enterprise. Notably we are not modeling any type of manufacturing or research entities as this is just a retail store.   The detailed product schema is as follows:

&nbsp;

[<img class="alignnone size-full wp-image-362" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreProductSchema.png" alt="BikeStoreProductSchema" width="433" height="551" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/BikeStoreProductSchema.png)

### <span style="font-weight: 400;">Model</span>

A model represents a product, and may have one or more part numbers.  Coke and Coke Zero are two different models. Coke &#8211; 20 oz Bottle and Coke 12 oz can are two different part numbers of the Model &#8211; Coke.  Some systems do not contain an entity for model, but represent everything at the part number level. This of course, breaks <a href="https://en.wikipedia.org/wiki/First_normal_form">First Normal Form</a>.   In the bike store there are products that have variations on size / color, tied to detailed product information.  Using the Model entity allows a common profile of product information to be shared by one or more part numbers.

A key design need is to provide a structure for storing hundreds of disparate attributes for different types of products.  Simplistic systems create dozens of columns for an entity, which leads to highly sparse records, which are not efficient for storage, and requires schema changes when new attributes are needed.   The next best thing is to create a key/value table relationship in the form of Model->ModelDataFieldValue->ModelDataFieldtype.   This works well and I’ve done this in a number of scenarios.  However for this project I’m exploring the use of JSON for this information.  For each Model entity, I simply need to have a field to store the JSON data.  Here I’m actually using two.  One for category-specific fields, and one for model-specific fields.  For example, for bicycles, I want to record the groupset, frame type, frame fit, wheelset, tires, crank dimensions, etc.   This would apply to all bicycle categories.   However for Trek I might want to also record if the bike is eligible for ProjectOne, or a special rebate, etc.  

Ultimately the difference looks like this:

[<img class="alignnone size-full wp-image-363" src="http://brianvanderplaats.com/wp-content/uploads/2015/10/TraditionalTableRelationships.png" alt="TraditionalTableRelationships" width="685" height="151" />](http://brianvanderplaats.com/wp-content/uploads/2015/10/TraditionalTableRelationships.png)

vs.

```json
[
    {"Crucial BX100 250GB SSD":{
        "Capacity":  "250 GB",
        "Sequential Read":  "535 MBps",
        "Sequential Write":  "370 MBps"
		}
    },
    {"Samsung 850 EVO 250GB SSD":{
        "Capacity":  "250 GB",
        "Sequential Read":  "540 MBps",
        "Sequential Write":  "520 MBps"
		}
    }
]
```

(thanks to @kitroed for sending me SSD deal links while writing this post&#8230;)

<span style="font-weight: 400;">Advantages to storing in JSON</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Faster querying, removing additional joins</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Many front-end UI components work directly with JSON, don’t need to write code to process table results</span>
</li>

<span style="font-weight: 400;">Disadvantages to storing in JSON</span>

<li style="font-weight: 400;">
  Searching for keys matching a specific value cannot be done in a simple where clause
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Higher storage requirements (store JSON string name along with value)</span>
</li>

### <span style="font-weight: 400;">Part Number</span>

<span style="font-weight: 400;">Part Number or SKU represents the item that goes in inventory and sold to the customer.  A part number is identified by the manufacturer part number or the bike store inventory part number.  Most inventory systems will want to use a common / global part number scheme, since we have no control over what that manufacturer assigns in their inventory system. Note that a serial number is an instance of PartNumber at the inventory level for large items like bicycles.  </span>

<span style="font-weight: 400;">ex.</span>

<span style="font-weight: 400;"><strong>Model</strong>:  Gel Cork Tape</span>

<span style="font-weight: 400;"><strong>Invoice Description</strong>:  Gel Cork Tape &#8211; Red</span>

<span style="font-weight: 400;"><strong>Inventory Part Number</strong>: 10500</span>

<span style="font-weight: 400;"><strong>Manufacturer Part Number</strong>:  402500</span>

### <span style="font-weight: 400;">Manufacturer, Category, Status</span>

Manufacturer (or “make”) is essentially the label on the product e.g. Trek.  We don’t care if Trek actually manufactures their frame, or that Bontrager is simply a label that Trek owns &#8211; at the product level these are both manufacturer&#8217;s.  At the purchasing level, we may end up buying both bontrager and trek products from the same vendor/distributor.  

Category includes Road Bikes, Saddles, Grip Tape, etc.   Both Manufacturer and Category have JSON fields.  These fields store blank sets of string/value pairs.  Will be used by product entry screens to populate the related category / manufacturer specific fields that need to be filled out. 

Status is also standard, but note it doesn’t contain &#8220;out-of-stock&#8221;.  This is a dynamic status that should be determined by the inventory module, and could fluctuate on a daily basis, while the product is “active” the entire time.

### <span style="font-weight: 400;">Github, Visual Studio Database Projects</span>

I’ve implemented the database using Visual Studio 2015 database projects.  It includes initial sample data, and can be deployed repeatedly to the same database endpoint.  For the sample data, I’m using a merge approach, loading data from tab-delimited text files. This is the first time I’ve worked with database projects and overall I’m pretty happy with the tool.

I’ve deployed the current build to GitHub as [brianvp/bikestore-database](https://github.com/brianvp/bikestore-database).   As I work through my research areas I will update as needed.