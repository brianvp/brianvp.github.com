---
id: 462
title: 'Web API Series Part 2 - Basic Authentication'
date: 2016-01-20T20:31:51+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=462
permalink: /2016/01/20/web-api-series-part-2-basic-authentication/
categories:
  - Development
  - Projects
tags:
  - ASP.NET Identity
  - Basic Authentication
  - Entity Framework
  - REST
  - SQL Server
  - Web API
---
<span style="font-weight: 400;">In the previous <a href="http://brianvanderplaats.com/2015/12/10/web-api-series-part-1-basic-crud/">article </a>on Web API, I demonstrated simple CRUD operations.   In this article I implement a security scheme using Basic Authentication.   While investigating authentication options, I was first dismissive of Basic Authentication.   Sending credentials with every request that could easily be read doesn’t seem like a very secure approach.  Then I found a blog <a href="https://www.rdegges.com/2015/why-i-love-basic-auth/">post</a> by @rdegges that raised some good points.  Essentially, basic authentication can be a valid technique if you:</span>

  * <span style="font-weight: 400;">Require HTTPS</span>
  * <span style="font-weight: 400;">Use random key pairs for the API</span>
  * <span style="font-weight: 400;">Generate unique key pairs for each client application</span>
  * <span style="font-weight: 400;">Provide some mechanism of allowing clients to revoke / request their keys.   </span>

<span style="font-weight: 400;">However, if you’re planning on using the API as a back end for a front-end client, basic authentication is not a good choice.   Somewhere, your JavaScript will need to set the credentials on the API call, and to my knowledge, you can’t easily (possibly?) store that on the client in a secure manner.  In these cases you should stick to a session ID / token approach.  To put it another way, Basic Authentication should not be used unless the credentials can be stored in secure manner.  Later on I demonstrate this on the server end using a pass-thru API call, as if I was calling some third party API service on behalf of the client.  </span>

<span style="font-weight: 400;">Source is up on <a href="https://github.com/brianvp/webapi-examples/tree/master/WebAPIBasicAuthentication">Github</a></span>

## <span style="font-weight: 400;">Custom Filters for Authentication</span>

<span style="font-weight: 400;">Setting up basic authentication was slightly complicated.   The Web API / MVC frameworks do not support basic authentication as one of the default wizard options - you need to include custom code in your app.  You have the choice of either writing a custom HTTP Module, or using the newer Web API Filter approach. I ended up using this BasicAuthenticationAttribute filter from  <a href="http://www.asp.net/web-api/overview/security/authentication-filters">www.asp.net / Codeplex</a>.    </span>

Additionally, I also wanted to force HTTPS, and I was able to find another filter to copy from this www.asp.net [article](http://www.asp.net/web-api/overview/security/working-with-ssl-in-web-api).

## <span style="font-weight: 400;">Security Approach</span>

<span style="font-weight: 400;">The basic security scheme I used is as follows:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Only authenticated users may access the api routes</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">the connection must be HTTPS</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">restricted routes can only be accessed if the user is in the AdvancedUserRole</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Using ASP.NET Identity to store the API accounts and assign roles.  This is a separate database from the BikeStore database, and the schema was created automatically the first time I used the security context.  </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">the api account ID is a GUID without dashes, and the api “secret” is simply another GUID without dashes.  </span>
</li>

<span style="font-weight: 400;">To use the identity database, I created a new security context and database.   I added the users in the seed() method and updated using migrations.  </span>

## <span style="font-weight: 400;">Controllers</span>

<span style="font-weight: 400;">The controllers are essentially the same as in the prior <a href="http://brianvanderplaats.com/2015/12/10/web-api-series-part-1-basic-crud/">article</a></span>

<span style="font-weight: 400;">Each controller has been decorated with three attributes</span>

```csharp
namespace WebApiBasicAuthentication.Controllers
{
 [IdentityBasicAuthentication] // Enable Basic authentication for this controller.
 [RequireHttps]
 [Authorize] // Require authenticated requests.
 public class ModelsController : ApiController
 {
```

<span style="font-weight: 400;">For the restricted controller, you simply include the roles in the Authorize Attributes</span>

```csharp
namespace WebApiBasicAuthentication.Controllers
{
 [IdentityBasicAuthentication] // Enable Basic authentication for this controller.
 [RequireHttps]
 [Authorize (Roles = "AdvancedUserRole")] // Require authenticated requests + user in specified role
 public class PartNumbersController : ApiController
 {
 
```

<span style="font-weight: 400;">The new controller I added to this project was to demonstrate calling a third party service.  As I mentioned earlier, this would be a valid place to call an API using basic authentication, as the client has no knowledge of the credentials.   In this case the “third party service” is simply another controller in the project, but you get the idea.  </span>

```csharp
namespace WebAPIBasicAuthentication.Controllers
{
 public class PartNumberPassThruController : ApiController
 {
 public async Task<HttpResponseMessage> GetPartNumbers()
 {
 HttpResponseMessage response = null;
 string json = "";
 string apiKey = "7f5645122a634577a53dca81359138b6";
 string apiSecret = "3186b763ddb4405bbf0b0d0eac5892cf";

 using (var client = new HttpClient())
 {

 client.BaseAddress = new Uri(Request.RequestUri.GetLeftPart(UriPartial.Authority));
 client.DefaultRequestHeaders.Accept.Clear();
 client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
 client.DefaultRequestHeaders.Authorization =
 new AuthenticationHeaderValue(
 "Basic",
 Convert.ToBase64String(
 System.Text.ASCIIEncoding.ASCII.GetBytes(
 string.Format("{0}:{1}", apiKey, apiSecret)))
 );

 response = await client.GetAsync("api/PartNumbers");

 
 if (response.IsSuccessStatusCode)
 {
 return response;
 }
 }

 return Request.CreateResponse(HttpStatusCode.NotFound);
 } 
 }
}
```

<span style="font-weight: 400;">If the third party api returns JSON, you may choose to pass this directly to the client.  Or, you may choose to load into objects for further manipulation, filtering, etc.   </span>

## <span style="font-weight: 400;">Examples</span>

<span style="font-weight: 400;">The website root contains a number of sample tests</span>

### <span style="font-weight: 400;">Test #1 - Access the API directly via browser  </span>

[<span style="font-weight: 400;">https://localhost:44300/api/models</span>](https://localhost:44300/api/models)

<span style="font-weight: 400;">If you access this before other tests, you will be prompted to enter a username / password.  </span>

[<img class="alignnone  wp-image-463" src="http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthLoginPrompt.png" alt="WebApiBasicAuthLoginPrompt" width="244" height="184" />](http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthLoginPrompt.png)

<span style="font-weight: 400;">With basic authentication, the browser caches those credentials, and they will be sent with each request.  </span>

<span style="font-weight: 400;">Here’s what the header looks like (remember that the string after Basic is simply a base64 encoded version of the username/password):</span>

[<img class="alignnone size-full wp-image-464" src="http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthHeader.png" alt="WebApiBasicAuthHeader" width="939" height="208" />](http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthHeader.png)

### <span style="font-weight: 400;">Test #2 - Execute Client-Side API Call that is valid for any authenticated user</span>

```javascript
//General User
var apiKey = "5b2025f5c0c847b788545cce506ce6eb";
var apiSecret = "556038b0a8d943caaf1c1ddfab70f956";

$.ajax({
 type: "GET",
 url: "/api/models",
 dataType: "json",
 async: false,
 username: apiKey,
 password: apiSecret,
 data: "",
 success: function (data) {

 $('#ClientOutputGeneralAccess').html("");

 $('#ClientOutputGeneralAccess').append('All Models in the Database #1 \r\n');
 $.each(data, function (key, item) {
 $('#ClientOutputGeneralAccess').append(item.ModelId + " | " + item.Name + '\r\n');
 });

 },
 error: function (XMLHttpRequest, status, error) {

 $('#ClientOutputGeneralAccess').html("");
 $('#ClientOutputGeneralAccess').append(status + " " + error);
 }
 });
```

[<img class="alignnone  wp-image-465" src="http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthGeneralCall.png" alt="WebApiBasicAuthGeneralCall" width="321" height="164" />](http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthGeneralCall.png)

### <span style="font-weight: 400;">Test #3 - Execute Client-Side API Call that requires user to be in a Role</span>

The /api/PartNumbers route is restricted.  The first part of this test demonstrates what happens when a user not in the AdvancedUserRole tries to request this route.

```csharp
// general user
var apiKey = "5b2025f5c0c847b788545cce506ce6eb";
var apiSecret = "556038b0a8d943caaf1c1ddfab70f956";

$.ajax({
 type: "GET",
 url: "/api/PartNumbers",
 dataType: "json",
 async: false,
 username: apiKey,
 password: apiSecret,
 data: "",
 success: function (data) {
 $('#ClientOutputLimitedAccess').html("");
 $('#ClientOutputLimitedAccess').append('All Part numbers in the Database \r\n');
 $.each(data, function (key, item) {
 $('#ClientOutputLimitedAccess').append(item.PartNumberId + " | " + item.InvoiceDescription + '\r\n');
 });

 },
 error: function (XMLHttpRequest, status, error) {

 $('#ClientOutputLimitedAccess').html("");
 $('#ClientOutputLimitedAccess').append(status + " " + error);
 }
});
```

### [<img class="alignnone  wp-image-466" src="http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiRestrictedCallGeneralUser.png" alt="WebApiRestrictedCallGeneralUser" width="409" height="195" />](http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiRestrictedCallGeneralUser.png)

&nbsp;

Now the call is made using credentials in the AdvancedUserRole:

```javascript
//Advanced user
var apiKey = "7f5645122a634577a53dca81359138b6";
var apiSecret = "3186b763ddb4405bbf0b0d0eac5892cf";


$.ajax({
 type: "GET",
 url: "/api/PartNumbers",
 dataType: "json",
 async: false,
 username: apiKey,
 password: apiSecret,
 data: "",
 success: function (data) {
 $('#ClientOutputLimitedAccess').html("");

 $('#ClientOutputLimitedAccess').append('All Part numbers in the Database \r\n');
 $.each(data, function (key, item) {
 $('#ClientOutputLimitedAccess').append(item.PartNumberId + " | " + item.InvoiceDescription + '\r\n');
 });

 },
 error: function (XMLHttpRequest, status, error) {

 $('#ClientOutputLimitedAccess').html("");
 $('#ClientOutputLimitedAccess').append(status + " " + error);
 }
});
```

### [<img class="alignnone  wp-image-467" src="http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiRestrictedCallAdvancedUser.png" alt="WebApiRestrictedCallAdvancedUser" width="378" height="259" />](http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiRestrictedCallAdvancedUser.png)
  
<span style="font-weight: 400;">Test #4 - Execute a Server-Side API call</span>

The final test is for the pass-thru call.  Notice there is no API key / credentials in this call.  The restricted /api/partnumbers route is being accessed on the server.

```javascript
// server side call
$.ajax({
type: "GET",
url: "/api/PartNumberPassThru",
dataType: "json",
async: false,
data: "",
success: function (data) {
 $('#ServerOutput').html("");
 
 
 $('#ServerOutput').append('All Part numbers in the Database \r\n');
 $.each(data, function (key, item) {
 $('#ServerOutput').append(item.PartNumberId + " | " + item.InvoiceDescription + '\r\n');
 });

},
error: function (XMLHttpRequest, status, error) {

 $('#ServerOutput').html("");
 $('#ServerOutput').append(status + " " + error);
}
}); 
```

[<img class="alignnone  wp-image-468" src="http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthServerCall.png" alt="WebApiBasicAuthServerCall" width="362" height="224" />](http://brianvanderplaats.com/wp-content/uploads/2016/01/WebApiBasicAuthServerCall.png)