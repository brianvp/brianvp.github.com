---
id: 434
title: 'Web API Series Part 1 - Basic CRUD'
date: 2015-12-10T22:52:59+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=434
permalink: /2015/12/10/web-api-series-part-1-basic-crud/
categories:
  - Development
  - Projects
tags:
  - Entity Framework
  - jQuery
  - REST
  - SQL Server
  - Web API
---
<span style="font-weight: 400;">In this article I demonstrate how to set up a Web API Service to perform simple CRUD operations.   Web API is a framework within ASP.NET that allows you to create RESTful services over HTTP.  The primary use case is for allowing communication from JavaScript front ends using JSON.  </span><span style="font-weight: 400;">For this example I did not include any authentication / security, which is obviously required for a production application.  </span>

<span style="font-weight: 400;">As usual the full source code is up at <a href="https://github.com/brianvp/webapi-examples/tree/master/WebAPIBasicCRUD">github </a></span>

## <span style="font-weight: 400;">RESTful Principles  </span>

<span style="font-weight: 400;">Before creating a Web API Service, you should understand the principles & conventions behind REST.  If you are coming from a traditional client/server background, these are not obvious, and you should take some time to understand the differences.  REST, or </span>[<span style="font-weight: 400;">Representational State Transfe</span>](https://en.wikipedia.org/wiki/Representational_state_transfer)<span style="font-weight: 400;">r, is not a technology, protocol, or standard, it is a software architectural pattern.  It describes a way of interacting with resources through HTTP.  A RESTful API is an api that adheres to the REST principles.  </span>

<span style="font-weight: 400;">The key principles are:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Tables/entities/things are defined as </span><i><span style="font-weight: 400;">resources</span></i>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Resources are requested or interacted with using standard HTTP verbs:</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">GET - retrieve a resource</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">POST - create a new resource</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">PUT - update a resource</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">PATCH - partially update a resource</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">DELETE - remove a resource</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Clients do not concern themselves with data storage / retrieval (i.e. the client doesn’t know that entity framework is used to query the database or the database is SQL Server)</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Stateless Design</span>
</li>

<span style="font-weight: 400;">The most important concept to understand is that when calling a service, we shouldn’t think in terms of executing some method, but that we are requesting or performing some action on a </span><span style="font-weight: 400;">resource</span><span style="font-weight: 400;">.  For the most part, an resource is largely the same as an entity defined in Entity Framework, so if you are familiar with that you shouldn’t have too much trouble understanding this concept.  This means that resources are nouns, not verbs.  Customers is a resource, Orders is a resource, GetCustomerOrderHistory is \*not\* a resource.  It’s actually two resources - one Customers, and two, order history.  a RESTful path looks like this: (Note that the convention is to pluralize resource names, just like in Entity  Framework.  )</span>

```
/api/Customers/21312/OrderHistory/
```

<span style="font-weight: 400;">As I’ll discuss later, not everything neatly maps to a resource, and that’s OK,  Pick something that makes sense, and stick with it for your application.  </span><span style="font-weight: 400;">This </span>[<span style="font-weight: 400;">blog post</span>](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)<span style="font-weight: 400;"> discusses several best practices for creating RESTful services.  Even if you aren&#8217;t publishing a public API, you should approach your API design with the same level of care as you would your front-end layout.  Your API should not be a large collection of methods, but a carefully laid out hierarchy of resources and actions.  </span>

## <span style="font-weight: 400;">Create the project & Default options</span>

<span style="font-weight: 400;">Setting up a Web API Project is almost identical to setting up an MVC project:</span>

[<img class="alignnone size-full wp-image-435" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/CreateNewWebAPIProject.png" alt="CreateNewWebAPIProject" width="702" height="302" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/CreateNewWebAPIProject.png)

&nbsp;

[<img class="alignnone size-full wp-image-436" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/WebAPIProjectTemplate.png" alt="WebAPIProjectTemplate" width="795" height="616" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/WebAPIProjectTemplate.png)

The default project templates will include an MVC Homepage, various authentication controllers, bootstrap, jquery, etc.

## <span style="font-weight: 400;">Setting up Database / Entities</span>

<span style="font-weight: 400;">For this example I’m using Entity Framework  Code First connected to the <a href="https://github.com/brianvp/bikestore-database">BikeStore database</a></span>

<span style="font-weight: 400;">Once you’ve created your core entities, you need to modify them slightly to work with WebAPI.  If any of your entities use navigation properties, WebAPI will attempt to serialize these properties into a JSON bundle.  For example our Model class has a collection of PartNumbers.  When one of our WebAPI GET methods returns a model, the serializer will attempt to expand all related part numbers for a given model, which will cause a large number of database queries & processing on the web server.  In most cases this is not what we want, so we have the following options:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Turn off navigation properties by marking as non-virtual (remove the virtual keyword).  This works but then we can’t use navigation on the server when we want it.</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Create Data Transfer Objects (DTO’s).  Create a custom object with all the properties you intended to send to the client (serialize).  This works well, and has the benefit of allowing us to create composite results (columns from multiple tables/entities), but if you just want to return a Model entity/record, it seems like unnecessary duplication.</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Return anonymous types.  You can simply write a linq query that returns the fields you want.  But this doesn’t help your API documentation, which needs to be supplied a type - either the source entity or some sort of Data Transfer object.</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Decorate your Navigation Properties with the [JsonIgnore] Attribute.   This doesn’t work if you need to return an XML result, but for JSON it works perfectly.  This prevents the serializer from attempting to expand your navigation properties when returning a Model, and is the solution I chose for this project.   </span>
</li>

## <span style="font-weight: 400;">Add WebAPI Controllers </span>

<span style="font-weight: 400;">This part is simple - the default scaffolding is very good.  Right-click on the controllers folder and select Add->Controller:</span>

[<img class="alignnone size-full wp-image-437" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/AddNewWebApiController.png" alt="AddNewWebApiController" width="951" height="650" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/AddNewWebApiController.png)

<span style="font-weight: 400;">Choose the “Web API 2 Controller with actions, using Entity Framework” options</span>

<span style="font-weight: 400;">You will then need to select an entity framework class & context:</span>

[<img class="alignnone size-full wp-image-438" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/AddNewWebApiControllerSetName.png" alt="AddNewWebApiControllerSetName" width="596" height="231" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/AddNewWebApiControllerSetName.png)

## <span style="font-weight: 400;">Reviewing the Default Scaffolding </span>

<span style="font-weight: 400;">By default, the generated code almost covers every basic CRUD operation we would want to do on the model.  This next section shows the code for each.  </span>

### <span style="font-weight: 400;">SELECT</span>

```csharp
public IQueryable<Model> GetModels()
 {

 return db.Models;

 }

 // GET: api/Models/5
 [ResponseType(typeof(Model))]
 public IHttpActionResult GetModel(int id)
 {
 Model model = db.Models.Find(id);

 if (model == null)
 {
 return NotFound();
 }

 return Ok(model);
 }

```

### <span style="font-weight: 400;">CREATE</span>

```csharp
 // POST: api/Models
 [ResponseType(typeof(Model))]
 public IHttpActionResult PostModel(Model model)
 {
 if (!ModelState.IsValid)
 {
 return BadRequest(ModelState);
 }

 db.Models.Add(model);
 db.SaveChanges();

 return CreatedAtRoute("DefaultApi", new { id = model.ModelId }, model);
 }
```

### <span style="font-weight: 400;">UPDATE</span>

```csharp
// PUT: api/Models/5
 [ResponseType(typeof(void))]
 public IHttpActionResult PutModel(int id, Model model)
 {
 if (!ModelState.IsValid)
 {
 return BadRequest(ModelState);
 }

 if (id != model.ModelId)
 {
 return BadRequest();
 }

 db.Entry(model).State = EntityState.Modified;

 try
 {
 db.SaveChanges();
 }
 catch (DbUpdateConcurrencyException)
 {
 if (!ModelExists(id))
 {
 return NotFound();
 }
 else
 {
 throw;
 }
 }

 return StatusCode(HttpStatusCode.NoContent);
 }
```

### <span style="font-weight: 400;">DELETE</span>

```csharp
// DELETE: api/Models/5
 [ResponseType(typeof(Model))]
 public IHttpActionResult DeleteModel(int id)
 {
 Model model = db.Models.Find(id);
 if (model == null)
 {
 return NotFound();
 }

 db.Models.Remove(model);
 db.SaveChanges();

 return Ok(model);
 }
```

&nbsp;

<span style="font-weight: 400;">To test this code, simply run the solution, and type /api/models after localhost:XXXX in the address bar. You should see a JSON result for each model in the database.  Later on we will create a simple jQuery test framework to call the web methods.  </span>

[<img class="alignnone size-full wp-image-439" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/TestApiModelsPath.png" alt="TestApiModelsPath" width="596" height="304" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/TestApiModelsPath.png)

## <span style="font-weight: 400;">Examining WebAPI Routing</span>

<span style="font-weight: 400;">In the previous example, we saw that the /api/models path returned a JSON result.  Looking at our ModelsController class, we see that this most likely called the GetModels() method, but how does this actually work?  </span>

<span style="font-weight: 400;">The main reason this looks simple is the heavy use of default routing conventions and use of HTTP methods.  </span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">GET - </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/api/models</span></a>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">GET </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/</span></a><span style="font-weight: 400;">api/models/10000</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">POST </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/</span></a><span style="font-weight: 400;">api/models/</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">PUT /</span><span style="font-weight: 400;">api/models/10000</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">DELETE /</span><span style="font-weight: 400;">api/models/10000</span>
</li>


<span style="font-weight: 400;">In WebApiConfig.cs, the standard route table is</span>

```csharp
config.Routes.MapHttpRoute(
 name: "DefaultApi",
 routeTemplate: "api/{controller}/{id}",
 defaults: new { id = RouteParameter.Optional }
 ); 
```

<span style="font-weight: 400;">When a HTTP GET request is received for </span>[<span style="font-weight: 400;">/api/models</span>](http://mysite.com/api/models)<span style="font-weight: 400;">, this route is hit:</span>

<li style="font-weight: 400;">
  <b>api/</b><span style="font-weight: 400;">{controller}/{id} -> </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/</span><b>api/</b><span style="font-weight: 400;">models</span></a>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">api/{</span><b>controller</b><span style="font-weight: 400;">}/{id} -> </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/api</span><b>/models</b></a> <span style="font-weight: 400;">- this takes the “models” string, adds “Controller” to look for an APIController class named “ModelsController”.  Since this class exists, the class takes over the rest of the request.</span>
</li>

<span style="font-weight: 400;">Ultimately </span>[<span style="font-weight: 400;">/api/models</span>](http://mysite.com/api/models) <span style="font-weight: 400;">maps to the GetModels() method.  How does this work inside controller?  </span>

<span style="font-weight: 400;">First, look for any public method starting with “Get”.  Next, find a Get method that matches the Request parameters *exactly*.  Since the calls was for /api/models and not api/models/10000, there are no parameters, so the parameterless GetModels() is executed.  If the call was for api/models/10000,  then the GetModel(int id) method is called.</span>

<span style="font-weight: 400;">This is very interesting as the full method name has no bearing on the action taken.  We could call it GetModels(), GetSomeModels(), whatever.  But we can’t call it ModelList(), and we can’t have two methods with the same list of parameters.  In the first case we will get an error:</span>

**{&#8220;message&#8221;:&#8221;the requested resource does not support http method &#8216;GET&#8217;.&#8221;}**

<span style="font-weight: 400;"> and the second will be an error that multiple routes are available:</span>

**{&#8220;message&#8221;:&#8221;An error has occurred.&#8221;, &#8220;ExceptionMessage&#8221;:&#8221;Multiple actions were found that match the request:&#8230;&#8221;}**

<span style="font-weight: 400;">It’s also interesting as the URL for getting a model and updating a model are exactly the same.  The chosen controller method is based on the HTTP request method:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">GET </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/</span></a><span style="font-weight: 400;">api/models/10000 maps to GetModel(int id)</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">PUT </span><a href="http://mysite.com/api/models"><span style="font-weight: 400;">/</span></a><span style="font-weight: 400;">api/models/10000 maps to PutModel(int id, Model model) (note the model is populated via the request body, not the URL)</span>
</li>

<span style="font-weight: 400;">This is getting to be a little long, but the reason I bring this up is that for a traditional client/server developer used to simple business classes etc, this is approach is counter intuitive.</span>

## <span style="font-weight: 400;">Search Operations</span>

<span style="font-weight: 400;">Before looking at the test framework, there is one important type of operation we need to review - searching!  In our example search, we want to get back a list of part numbers by supplying one or more parameters.  However, we know that the query needs to join both PartNumbers and Models, so which controller does it go in?  There are two basic schools of thought here. The first way is to determine which entity is most directly related to the result set, and create a sub-resource/path.  In our case, PartNumber is a reasonable choice, so we will add a custom route/method to the PartNumber controller.  The second school of thought is to treat a search as a top-level resource itself.   Both methods are shown below.  </span>

<span style="font-weight: 400;">In both cases, I recommend using Web API </span>[<span style="font-weight: 400;">attribute routing</span>](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)<span style="font-weight: 400;">.  Simply stated, attribute routing explicitly decorates controller methods for the HTTP Method and URL Path they are expected to process.  For example, the paths below do not fit the standard route pattern of “api/{controller}/<id></span>

<span style="font-weight: 400;">/api/search/partNumber<br /> </span><span style="font-weight: 400;">/api/search/model<br /> </span><span style="font-weight: 400;">/api/partnumber/search</span>

<span style="font-weight: 400;">Rather than coming up with a new custom route pattern, you can simply add:</span>

<span style="font-weight: 400;">[Route(“api/search/partnumber”]<br /> </span><span style="font-weight: 400;">[HttpGet]</span>

<span style="font-weight: 400;">above SearchController.cs.PartNumberSearch()</span>

<span style="font-weight: 400;">The problem is that the standard routing expects each controller to implement one set of GET methods that are related i.e. variations on ModelSearch, not ModelSearch, PartNumberSearch, Categorysearch, etc.  This could work if the search parameters are all different, but if anything that would just be a coincidence.  The other approach of course is to simply create a new controller class for each type of search but that is just likely to create dozens of one-off controllers which doesn’t seem like good design…</span>

### <span style="font-weight: 400;">Parameters</span>

<span style="font-weight: 400;">For many types of searching there are often parameters that don’t fit the RESTful pattern of /resource/id.  Fortunately, you can simply use good old query strings e.g. /resource?Param1=&Param2=&Param3= etc.  One special consideration however- for your controller methods, you should always set a default value (usually null) foreach parameter.  For example:</span>

<span style="font-weight: 400;">/api/search/PartNumbers?ModelName=Tape will *not* map to PartNumberSearch(string modelName, string partNumberName).   For this to work you would need to change the URI to /api/search/PartNumbers?ModelName=Tape?PartNumberName=.    You can get around this by changing the controller method to: PartNumberSearch(string modelName = null, string partNumberName = null)</span>

### <span style="font-weight: 400;">Search as a Sub-Resource</span>

<span style="font-weight: 400;">/api/PartNumbers/Search?modelName=Tape</span>

```csharp
[Route("api/PartNumbers/Search")]
 [ResponseType(typeof(ProductSearchResultDTO))]
 public IHttpActionResult GetPartNumberSearch(string modelName = null, string partNumberName = null)
 {

 var query = from md in db.Models
 join pn in db.PartNumbers
 on md.ModelId equals pn.ModelId
 join ct in db.Categories
 on md.CategoryId equals ct.CategoryId
 where ( md.Name.Contains(modelName)|| modelName == null ) && 
 (pn.Name.Contains(partNumberName) || partNumberName == null )
 select new ProductSearchResultDTO { ModelId = md.ModelId, ModelName = md.Name, PartNumberName = pn.Name, InventoryPartNumber = pn.InventoryPartNumber, ListPrice = pn.ListPrice, CategoryName = ct.Name };

 return Ok(query);
 }
```

### <span style="font-weight: 400;">Search as a Top-Level Resource</span>

<span style="font-weight: 400;">/api/search/PartNumbers?modelName=Tape</span>

```csharp
[Route("api/Search/PartNumbers/")]
 [HttpGet]
 [ResponseType(typeof(ProductSearchResultDTO))]
 public IHttpActionResult PartNumberSearch(string modelName = null, string partNumberName = null) 
 {
 var query = from md in db.Models
 join pn in db.PartNumbers
 on md.ModelId equals pn.ModelId
 join ct in db.Categories
 on md.CategoryId equals ct.CategoryId
 where (md.Name.Contains(modelName) || modelName == null) &&
 (pn.Name.Contains(partNumberName) || partNumberName == null)
 select new ProductSearchResultDTO { ModelId = md.ModelId, ModelName = md.Name, PartNumberName = pn.Name, InventoryPartNumber = pn.InventoryPartNumber, ListPrice = pn.ListPrice, CategoryName = ct.Name };

 return Ok(query);
 }
```

<span style="font-weight: 400;">For this example, I’ve left both routes in the solution, but if I had to choose, I think the sub-resource works well.  Most entities will need some sort of basic search, and I’d prefer to keep these in related controllers.   </span>

## <span style="font-weight: 400;">Test Framework</span>

<span style="font-weight: 400;">To test, I’ve set up a very simple jquery client that simply runs when you hit the site home page.  </span>

<span style="font-weight: 400;">The code is straightforward.  N</span><span style="font-weight: 400;">ote that when a resource is requested that does not exist, the HTTP return type changes from 200 to 404 not found. This is different than a traditional business/service layer that will usually return an empty or null result.  </span>

&nbsp;

```csharp
$(document).ready(function () {

 //call to prevent later getJSON calls from firing before previous ones
 //obviously not something to do in a production app, as this blocks
 //the user from browser interaction...
 $.ajaxSetup({
 async:false
 })

 //select all Models
 $.getJSON('/api/models', function (data) {
 $('#console').append('All Models in the Database \r\n');
 $.each(data, function (key, item) {
 $('#console').append(item.ModelId + " | " + item.Name + '\r\n');
 });
 });

 //filtered query
 $.getJSON('/api/Search/PartNumbers?modelName=tape', function (data) {
 $('#console').append('All Bar Tape Products in the Database \r\n');
 $.each(data, function (key, item) {
 $('#console').append(item.ModelId + " | " + 
 item.ModeName + " | " + 
 item.PartNumberName + " | " +
 item.InventoryPartNumber + " | " +
 item.ListPrice + " | " +
 item.CategoryName + " | " + '\r\n');
 });
 });

 

 // create a new model
 var modelId = 0;
 var newModel = { Name: "Domane 5.2", ListPrice: 3499.99 };

 $.ajax({
 type: "POST",
 data: JSON.stringify(newModel),
 url: "/api/models",
 contentType: "application/json"
 }).done(function (result) {

 modelId = result.ModelId;

 });
 
 // update model
 var modelToUpdate = null;
 $.getJSON('/api/Models/' + modelId, function (data) {
 modelToUpdate = data;
 });

 modelToUpdate.Features = "500 Series OCLV Frame";

 $.ajax({
 type: "PUT",
 data: JSON.stringify(modelToUpdate),
 url: "/api/models/" + modelId,
 contentType: "application/json"
 })

 //select single model
 $.getJSON('/api/models/' + modelId, function (data) {
 $('#console').append('Selected Model: \r\n');
 $('#console').append(data.ModelId + " | " + data.Name + '\r\n');
 
 });

 // delete model
 $.ajax({
 type: "DELETE",
 url: "/api/models/" + modelId,
 contentType: "application/json"
 })

 //select single model (this will be a 404 error as the model was deleted in previous operation)
 $.getJSON('/api/models/' + modelId, function (data) {
 $('#console').append('Selected Model: \r\n');
 $('#console').append(data.ModelId + " | " + data.Name + '\r\n');

 });
 
 } );
```

Here is the resulting network activity:

[<img class="alignnone size-full wp-image-446" src="http://brianvanderplaats.com/wp-content/uploads/2015/12/WebAPICRUDNetworkActivity.png" alt="WebAPICRUDNetworkActivity" width="749" height="224" />](http://brianvanderplaats.com/wp-content/uploads/2015/12/WebAPICRUDNetworkActivity.png)

## <span style="font-weight: 400;">Final Words</span>

<span style="font-weight: 400;">Creating a simple wrapper over entity framework classes is almost trivial in Web API.  With the extra time savings not wiring up dozens of methods, you can put thought into your API organization, and how to deal with those hybrid resources that don’t strictly fit RESTful conventions&#8230; </span>

<span style="font-weight: 400;">The best thing I see about Web API is being able to take advantage of new front end-tech like AngularJS without having to re-invent your back end tools.  If you are comfortable working in .NET / SQL Server, you don’t see a need to run out and learn Node.js as well.   From an infrastructure perspective that’s a huge win, as deploying a Web API site is virtually no different than a web forms or MVC site.  </span>