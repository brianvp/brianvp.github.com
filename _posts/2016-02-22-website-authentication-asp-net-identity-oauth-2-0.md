---
id: 478
title: 'Website Authentication &#8211; ASP.NET Identity &#038; OAuth 2.0'
date: 2016-02-22T20:18:21+00:00
author: brianvp
layout: post
guid: http://brianvanderplaats.com/?p=478
permalink: /2016/02/22/website-authentication-asp-net-identity-oauth-2-0/
categories:
  - Development
  - Projects
tags:
  - AngularJS
  - ASP.NET Identity
  - ASP.NET MVC
  - Entity Framework
  - Oauth 2.0
  - SQL Server
  - Web API
---
<span style="font-weight: 400;">In the next few posts, I will be exploring different authentication options for a simple front end/ back end app.  What I would like to demonstrate is the following:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Secure access to a server generated web page &#8211; in this case an ASP.NET MVC page</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Secure access to a web api call that will be used by a client side program, in this case AngularJS</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Demonstrate different levels of security using Security Roles</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Provide a mechanism for a user to log in</span>
</li>

<span style="font-weight: 400;">To do this I’ve set up a “Model Manager” website that has the following structure</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">/ &#8211; Home Page</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">/ModelEditor &#8211; List Of Models &#8211; ASP.NET MVC</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/Details &#8211; View Model Info</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/Edit &#8211; modify a model</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/Create &#8211; add a new model</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">/ModelEditorAngular &#8211; List of Models &#8211; AngularJS</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/<model #> &#8211; View/Edit/Create individual model</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">/Api &#8211; ASP.NET Web API</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/Models</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/PartNumbers</span>
    </li>
  </ul>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">/Account</span> <ul>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">/Login</span>
    </li>
  </ul>
</li>

<span style="font-weight: 400;">Basic security is as follows</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Anyone can hit the home page</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Authenticated users can view the List of models or view models (both MVC and Angular)</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Authenticated users in the “ModelEditorRole” can edit models or create new models</span>
</li>

<span style="font-weight: 400;">Other details</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">I’m testing these sites running locally, as well as on a Azure deployed service</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The application is built on the </span><a href="https://github.com/brianvp/bikestore-database"><span style="font-weight: 400;">BikeStore </span></a><span style="font-weight: 400;">database</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">I’m using a standalone database for user and role security</span>
</li>

<span style="font-weight: 400;">In the first example, I’ve explored the ASP.NET Identity authentication mechanisms, as well as OAuth 2.0. </span>
  
<span style="font-weight: 400;">Source code is up at <a href="https://github.com/brianvp/authentication-examples/tree/master/ModelManagerOAuthIndividual">github</a></span>

## <span style="font-weight: 400;">ASP.NET Identity Authentication</span>

<span style="font-weight: 400;"><a href="http://www.asp.net/identity/overview/getting-started/introduction-to-aspnet-identity">ASP.Net Identity</a> is the current out-of-the-box solution for ASP.NET website security.  It is built on Entity Framework, and gives you a lot of flexibility in setting things up.  With the default scaffolding that is part of the standard asp.net project template, it is very easy to provide a login mechanism for your users.   </span>

<span style="font-weight: 400;">To use ASP.NET Identity, you need the following</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">a login interface &#8211; provide by the AccountController / Account Views &#8211; I didn’t change anything from the default template</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">a SQL database & valid database context.     </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Authentication configuration in Startup.Auth.cs &#8211; not much was changed here.  </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">a mechanism to add and assign roles &#8211; I manually assigned roles in the database directly.</span>
</li>

<span style="font-weight: 400;">Setting up the database & context proved to be the most challenging part of this process.  The first step is deciding whether or not you want to use the ASP.NET Identity classes as-is, or to subclass.   If you choose to subclass, you can add custom properties to the User class that will be present when the tables are created.   The convention I’ve seen is to put this customization in an IdentityModels.cs  here’s what mine looks like:</span>

<pre class="brush: csharp; title: ; notranslate" title="">public class ApplicationUser : IdentityUser
{
 public async Task&lt;ClaimsIdentity&gt; GenerateUserIdentityAsync(UserManager&lt;ApplicationUser&gt; manager)
 {
 // Note the authenticationType must match the one defined in CookieAuthenticationOptions.AuthenticationType
 var userIdentity = await manager.CreateIdentityAsync(this, DefaultAuthenticationTypes.ApplicationCookie);
 // Add custom user claims here
 return userIdentity;
 }
}

public class ApplicationDbContext : IdentityDbContext&lt;ApplicationUser&gt;
{
 public ApplicationDbContext()
 : base("DefaultConnection", throwIfV1Schema: false)
 {
 }

 public static ApplicationDbContext Create()
 {
 return new ApplicationDbContext();
 }
}
</pre>

<span style="font-weight: 400;">The next option is to determine if you want to let the database creation happen automatically, or to enable migrations.   I chose to enable migrations so I could create a Seed() method for setting up some default users & adding the ModelEditorRole.   Frankly I’m not a fan of the implict table creation.  During testing, I ended up creating the security tables inside the BikeStore database by accident.  And while it’s kind of neat that the first time someone logs in the tables get created automatically, why would you ever have a real application deployed this way?  </span>

[<img class="alignnone size-full wp-image-480" src="http://brianvanderplaats.com/wp-content/uploads/2016/02/ModelManagerSecurityDb.png" alt="ModelManagerSecurityDb" width="792" height="349" />](http://brianvanderplaats.com/wp-content/uploads/2016/02/ModelManagerSecurityDb.png)

<span style="font-weight: 400;">Once up and running, the logon process is very simple:</span>

[<img class="alignnone size-full wp-image-481" src="http://brianvanderplaats.com/wp-content/uploads/2016/02/UserLogin.png" alt="UserLogin" width="641" height="405" />](http://brianvanderplaats.com/wp-content/uploads/2016/02/UserLogin.png)

[<img class="alignnone size-full wp-image-482" src="http://brianvanderplaats.com/wp-content/uploads/2016/02/User_logged_in.png" alt="User_logged_in" width="694" height="204" />](http://brianvanderplaats.com/wp-content/uploads/2016/02/User_logged_in.png)

<span style="font-weight: 400;">Here’s what a user looks like</span>

[<img class="alignnone size-full wp-image-483" src="http://brianvanderplaats.com/wp-content/uploads/2016/02/user_record.png" alt="user_record" width="974" height="48" />](http://brianvanderplaats.com/wp-content/uploads/2016/02/user_record.png)

## <span style="font-weight: 400;">OAuth 2.0 Authentication</span>

<span style="font-weight: 400;">The identity system is an effective choice to be sure, but what if you don’t want to manage user passwords, or you want the user to be able to sign in with their google, facebook, etc account?  The answer is OAuth.   OAuth is an open </span>[<span style="font-weight: 400;">standard</span>](http://tools.ietf.org/html/rfc6749) <span style="font-weight: 400;">for authorization, that is supported by all the major providers.  A website using OAuth can delegate user authentication to the service hosting the user’s account (i.e. Facebook).  the OAuth standard defines how this should work.  Digital Ocean has an excellent </span>[<span style="font-weight: 400;">overview</span>](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2) <span style="font-weight: 400;">on Oauth 2.0. </span>

<span style="font-weight: 400;">To use OAuth, you must first choose a provider, and register your application.  You must provide your application name, website, and a redirect URI.   The provider will then provide you with a Client ID and Client Secret, which will be used by your application to authenticate itself with the provider.  </span>

<span style="font-weight: 400;">Once configured, the application follows a special authentication workflow, generally initiated when the user tries to log in to your application.</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Application redirects the user to the configured Authorization Code Link.  This link contains</span> <ol>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">The path to the provider’s Authorization endpoint e.g. the Google Sign in Page</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">The websites client_id</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">a callback URL for the website &#8211; google will redirect back to this URL after authentication</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">a response_type &#8211; the type of “Grant” the application is requesting &#8211; which is typically for an “authorization code”</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Scope: either a read or write level of access</span>
    </li>
  </ol>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">User Logs in to service on the *provider’s* website, not our application.  </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The provider redirects to the callback URL, sending an authorization code</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Application requests an Access token.  Must send</span> <ol>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">authorization code (from previous step)</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">client id</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">client secret</span>
    </li>
  </ol>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Access receives an access token in JSON form, which contains</span> <ol>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">token type, usually “bearer”</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">expiration</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">Whether or not this is a refresh token or not</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">scope</span>
    </li>
    <li style="font-weight: 400;">
      <span style="font-weight: 400;">claims about the user e.g. name is Brian, email is </span><a href="mailto:Brian@gmail.com"><span style="font-weight: 400;">Brian@gmail.com</span></a><span style="font-weight: 400;">, etc.  </span>
    </li>
  </ol>
</li>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">The application is now authorized.  </span>
</li>

<span style="font-weight: 400;">Your application is not limited to a single provider &#8211; you could allow the user to authenticate with Google, Twitter, and Facebook, all at the sametime.  However, you must decide if @brian is the same as </span>[<span style="font-weight: 400;">Brian@gmail.com</span>](mailto:Brian@gmail.com) <span style="font-weight: 400;">and </span>[<span style="font-weight: 400;">brian@facebook.com</span>](mailto:brian@facebook.com)<span style="font-weight: 400;">.  Of course you could choose to only allow a person to log in with one type of provider as well.  The point being once you have an authorization token, your application can interpret / use that token as you see fit.  </span>

<span style="font-weight: 400;">Out of the box ASP.Net supports Google, Twitter, Facebook, and Microsoft logins as authentication options.  I’m using Google for this example.   On the asp.net side, setting up google authentication is incredibly easy. You simply provide your google API key / secret in /app_start/Startup.auth.cs:</span>

<pre class="brush: csharp; title: ; notranslate" title="">app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
{
 ClientId = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
 ClientSecret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXX"
});
</pre>

<span style="font-weight: 400;">Big warning right here.  Do not actually put your API key in here!  This must come from a configuration file, and that file must not be checked into your source code.  Ever.   Exposing your API key means other people can steal it and incur charges against your account.  Hardcoding your API key means that if you need to change your credentials or they are shut off, your app simply stops working.   Scott Hanselman posted a good </span>[<span style="font-weight: 400;">article </span>](http://www.hanselman.com/blog/BestPracticesForPrivateConfigDataAndConnectionStringsInConfigurationInASPNETAndAzure.aspx)<span style="font-weight: 400;">on this.  Here I’m more or less following his advice.  Just remember to avoid checking the secret file into any type of source control, and when publishing the file, </span>[<span style="font-weight: 400;">encrypt </span>](https://msdn.microsoft.com/en-us/library/zhhddkxy.aspx)<span style="font-weight: 400;">it.  </span>

<pre class="brush: csharp; title: ; notranslate" title="">app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
{
 ClientId = System.Configuration.ConfigurationManager.AppSettings["GoogleClientId"],
 ClientSecret = System.Configuration.ConfigurationManager.AppSettings["GoogleClientSecret"] 
});
</pre>

<span style="font-weight: 400;">Publishing to Azure, the Google API Key is in the application settings configuration panel:</span>

&nbsp;

[<img class="alignnone size-full wp-image-484" src="http://brianvanderplaats.com/wp-content/uploads/2016/02/GoogleApiKeyStorage.png" alt="GoogleApiKeyStorage" width="554" height="180" />](http://brianvanderplaats.com/wp-content/uploads/2016/02/GoogleApiKeyStorage.png)

<span style="font-weight: 400;">The bulk of the work involves setting up your google account to allow your website to talk to google.  I’m following the steps outlined in this <a href="http://www.asp.net/mvc/overview/security/create-an-aspnet-mvc-5-app-with-facebook-and-google-oauth2-and-openid-sign-on#goo">article</a>.  I will show you the steps I took here, but keep in mind that Google can change their services at any time.  This is the main trade off to using a third party, you simply don’t have control over what they do on their end.  The good news is that 3rd party authentication is really popular, and doesn’t look to be going away any time soon…  </span>

<span style="font-weight: 400;">Step 1: Create a new project</span>

<span style="font-weight: 400;">Step 2: Create credentials</span>

<span style="font-weight: 400;">Step 3: Define Acceptable Javascript origins and Redirect URI’s</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">This should include your local test environment e.g. </span><a href="https://localhost:44300"><span style="font-weight: 400;">https://localhost:44300</span></a>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">any staging servers you might have</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">your production website address</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Include HTTP and HTTPS addresses separately.   Ideally you should be using HTTPS only…</span>
</li>

<span style="font-weight: 400;">Step 4: Enable the Google + API </span>

<span style="font-weight: 400;">This is not enabled by default, and authentication will fail if it is not</span>

<span style="font-weight: 400;">Step 5: Get your provided ClientID / Secret and add them to your application</span>

<span style="font-weight: 400;">After configuration, you may attempt to log into the site with your google account.  </span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;"> click on Logon</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The site redirects you to the google logon.   </span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Chose your google account, and allow google to send your information to the website</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The site will prompt you to register.   This will create a record in the identity database with your google account.  Note that your password is blank &#8211; your website never see’s your account credentials.</span>
</li>

## <span style="font-weight: 400;">Fixing Account Redirection</span>

<span style="font-weight: 400;">The default behavior of the identity system is to redirect the user to the logon page whenever they hit a route that requires authorization.  This is very useful for someone who is not logged in, but what happens when someone not in the ModelEditorRole attempts to edit a model?  They get redirected to the logon page again.   This is confusing for the user, since they’ve already logged in.   </span>

<span style="font-weight: 400;">The solution for this is relatively simple, but disappointing.  I can’t imagine a production application that would use the standard redirection as-is, so having to create a custom filter just to fix this issue is irritating.  </span>

<span style="font-weight: 400;">From this Stack overflow </span>[<span style="font-weight: 400;">answer</span>](http://stackoverflow.com/a/5239540/24892)<span style="font-weight: 400;">:</span>

<pre class="brush: csharp; title: ; notranslate" title="">public class AuthorizeRedirectMVCAttribute : System.Web.Mvc.AuthorizeAttribute
{
 protected override void HandleUnauthorizedRequest(AuthorizationContext filterContext)
 {
 base.HandleUnauthorizedRequest(filterContext);

 if (filterContext.RequestContext.HttpContext.User.Identity.IsAuthenticated)
 {
 filterContext.Result = new RedirectResult("~/Account/AccessDenied");
 }
 }
}
</pre>

<span style="font-weight: 400;">Then, change your decorations from [Authorize] to [AuthorizeRedirectMVC].</span>

<span style="font-weight: 400;">Additionally, you need to implement another [Authorize] filter for your API routes, as they do *not* use the same AuthorizeAttribute class. </span>

<pre class="brush: csharp; title: ; notranslate" title="">public class AuthorizeRedirectAPIAttribute : System.Web.Http.AuthorizeAttribute
{
 protected override void HandleUnauthorizedRequest(HttpActionContext actionContext)
 {
 base.HandleUnauthorizedRequest(actionContext);

 if (actionContext.RequestContext.Principal.Identity.IsAuthenticated)
 {
 actionContext.Response = actionContext.Request.CreateResponse(HttpStatusCode.Forbidden);
 }
 }
}
</pre>

## <span style="font-weight: 400;">Application Security</span>

<span style="font-weight: 400;">With the two authentication mechanisms available, let’s look at how the application works with these.</span>

<span style="font-weight: 400;">First, any time a user tries to access a route that requires a role or authentication, the application will automatically redirect to the logon screen.  </span>

<span style="font-weight: 400;">All WebApi Routes are set with a default Authorize and Require HTTPS Filter.  As mentioned in a previous blog, Service endpoints should never be served over unencrypted HTTP.</span>

<pre class="brush: csharp; title: ; notranslate" title="">public static class WebApiConfig
{
public static void Register(HttpConfiguration config)
{
 // Web API configuration and services
 config.Filters.Add(new AuthorizeRedirectAPIAttribute());
 config.Filters.Add(new RequireHttpsAttribute());
</pre>

<span style="font-weight: 400;">The ASP.NET MVC Controller for Models use a [Authorize] attribute for the entire controller,  and an Authorize(Roles) for the editing routes  </span>

<pre class="brush: csharp; title: ; notranslate" title="">namespace ModelManagerOAuthIndividual.Controllers
{
[AuthorizeRedirectMVC]
public class ModelEditorController : Controller
{

...

[AuthorizeRedirectMVC(Roles = "ModelEditorRole")]
public ActionResult Create()
{
...
</pre>

<pre></pre>

<span style="font-weight: 400;">The Angular portion is served by an MVC Controller that has the authorize attribute.  Within the Angular routes we do not have access to the [Authorize] filters.  To add granular security within the Angular app, I created an API Security Controller:</span>

<pre class="brush: csharp; title: ; notranslate" title="">public class SecurityController : ApiController
{
// /api/security/UserAuthenticated
[HttpGet]
[AllowAnonymous]
public bool UserAuthenticated()
{
 return RequestContext.Principal.Identity.IsAuthenticated;
}

// /api/security/UserInrole?rolename=ModelEditorRole
[HttpGet]
[AllowAnonymous]
public bool UserInRole(string roleName)
{
 return User.IsInRole(roleName);
}
}
</pre>

<span style="font-weight: 400;">Inside the Angular controller, we make the security check as the user attempts to access edit mode on the model detail controller</span>

<pre class="brush: jscript; title: ; notranslate" title="">$scope.enterEditMode = function () {

$http.get("/api/security/UserInRole?roleName=ModelEditorRole").success(
 function (data) {
 if (!data) {
 // note can't use $location.path here, as the MVC Access denied page is "outside" of the 
 // angular app, and $location.path always prepends a leading '/', so you need to use window.location.href
 window.location.href = $location.absUrl().split('ModelEditorAngular')[0] + 'account/accessdenied';
 }

 $scope.inEditMode = true;
 }); 
}
</pre>

<span style="font-weight: 400;">Remember that the actual API calls to access/edit the model data are still secured through our filters.  This is an important point &#8211; you need to implement security on both the client *and* the API calls.  </span>

## <span style="font-weight: 400;">Conclusion</span>

<span style="font-weight: 400;">ASP.NET give you a lot of flexible tools for configuring different authentication options.  For a public site, a good strategy would be to include the following security:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">A selection of 2-3 different OAuth providers &#8211; I’d go with Google, Facebook, and either Microsoft or Twitter</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">A way for the users to register a normal account on your site</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">A common security schema that links the different users together</span>
</li>

<span style="font-weight: 400;">It should be noted again that OAuth implementations are not static, and it’s very possible that OAuth is deprecated at some point in the future.  At the very least, you don’t want your site to stop working because google or facebook make a change! The other tradeoff is that if these services have downtime, it will impact your users.  But the number one reason for using OAuth is simply convenience for your users.  It’s more likely that a user will forget their login / password than google or facebook to be down.   I would argue that OAuth is </span>_<span style="font-weight: 400;">less</span>_ <span style="font-weight: 400;">secure &#8211; as it basically allows users to use the same key on many doors.  Implementing OAuth for a financial institution would be a bad idea &#8211; that’s basically giving the provider access to that account.   In general, most sites on the Internet would benefit from allowing OAuth.  </span>