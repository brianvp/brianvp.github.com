---
title: 'ADFS Authentication Adding Custom Claims'
date: 2017-01-19T00:00:00+00:00
author: brianvp
layout: post
permalink: /2017/01/19/ADFS-Authentication-Adding-Custom-Claims/
categories:
  - General
tags:
  - ADFS
  - ASP.NET Identity
---

While user authentication is a key component of AD FS, the returned user claims are powerful tools for client applications.  

- Common user information (Name, login, email) can be retrieved without code duplication in multiple applications
- Security roles can be added to the claims token, reducing roundtrips or querying security information from a database during application lifetimes
- Unique system fields / tokens / identifiers that would normally be stored in some global variable or hidden fields

There are two ways to add to the available claims:

1. Configure AD FS to send additional claims - this will be shown in a future post
2. Dynamically add claims during user sign-on

Before going into details here, a few high level points to keep in mind: 

1. Claim tokens are shared between all sites in a subdomain e.g `https://apps.example.com`
2. Claim tokens can expire (based on AD FS settings), or be removed by the user logging out.
3. Claims from the AD FS server can be removed at any time.  
4. Changes made to the claims will not affect users that have a current claims token. 

Thus, your application should never assume that a claim exists.

## Adding Roles to claims

An excellent usage of claims information is populating the application security roles the user has access to.  This is information that will be checked frequently during the applications lifetime, and querying a security database on each access is inefficient. 

To begin, you will need a security database with users/roles/groups etc.  At my company we use the venerable `ASPNETDB` database that has been part of the `asp.net membership` system for a long time. 

Adding the claims is done in the `ConfigureAuth()` method. In a vanilla configuration, this looks as follows:

```csharp
app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

app.UseCookieAuthentication(new CookieAuthenticationOptions());

app.UseWsFederationAuthentication(
    new WsFederationAuthenticationOptions
    {
        Wtrealm = realm,
        MetadataAddress = adfsMetadata
    });
```

Adding claims is done by specifying additional items in `WsFederationAuthenticationOptions`

```csharp
app.UseWsFederationAuthentication(
new WsFederationAuthenticationOptions
{
    Wtrealm = realm,
    MetadataAddress = adfsMetadata,
    Notifications = new WsFederationAuthenticationNotifications
    {
        SecurityTokenValidated = context =>
        {
            string accountName = "";
            foreach (var claim in context.AuthenticationTicket.Identity.Claims)
            {
                if (claim.Type == ClaimTypes.Upn)
                {
                    accountName = claim.Value;
                }
            }

            //Query roles for user from security database
            string[] stringRoles = { "TestRole1", "TestRole2", "TestRole3", "TestRole4" };
            var roles = stringRoles.Select(s => new { RoleName = s });

            //loop through returned roles, and add
            foreach (var role in roles)
            {
                context.AuthenticationTicket.Identity.AddClaim(
                                                    new Claim(ClaimTypes.Role, role.RoleName));
            }
            
            return Task.FromResult(0);
        }

    }
});
}

```

Basically we specify an event handler to fire as soon as the claims token exists.  Note while this is configured in `startup.auth.cs`, this code only runs when the user signs in. This means that if you change which roles the user has in the database, or in Active Directory, the user will not see these roles until they logout, and signs in again.

The first step is getting the user identity we will use to retreive user roles - the user name is a good option here, but you could use a number of claims (again, based on what the AD FS adminstrator configured)

 - nameidentfier - `brianvp`
 - upn - `brianvp@example.com`
 - emailaddress - `brian.vanderplaats@example.com`

 Next, you will take your chosen identifier and query your security database.

 Then, iterate through these roles, and add to the claims collection using the `AddClaim()` method.  Make sure to specify `ClaimTypes.Role`. The actual role name is simply a string. Note that if you specify a role twice, it will be added twice.  This should not be a problem with multiple applications, as again, this handler does not fire on app startup, only during the sign-in process.  Once the user is signed in to an application on a domain, they will not need to sign in to other applications on the domain.

## Using Roles

Now that the roles are added, you need to configure your application to use them.  The simplest way is to combine roles into `[Authorize]` tags on your controllers.  For example, you could lock down your editor pages by placing the role over create/edit controller actions, but not over the view actions:

```csharp
public ActionResult Details(int? id)
{
    ...
}

[HttpPost]
[ValidateAntiForgeryToken]
[Authorize(Roles = "ModelEditorRole")]
public ActionResult Create( Model model)
{
    ...
}

[Authorize(Roles = "ModelEditorRole")]
public ActionResult Edit(int? id)
{
    ...
}
```

Another way is to look at the user's current roles while performing an action inside a method. This can be done by iterating through the user's claims:

```csharp
bool inTestRole4 = false;
var claims = ((ClaimsIdentity)Thread.CurrentPrincipal.Identity).Claims;
foreach (var claim in claims)
{
    if (claim.Type == ClaimTypes.Role && claim.Value == "TestRole4")
    {
        inTestRole4 = true;
        break;
    }
}
```

Or, even easier, you can use the simple check:

```
if(User.IsInRole("TestRole4"))
{

}

```

## Adding custom claim types

Adding a custom claim is basically the same as adding a role claim.  Instead of specifying `ClaimTypes.Role`, you give it your own name / value:

```csharp
SecurityTokenValidated = context =>
{
    var accountName = context.AuthenticationTicket.Identity.Claims.Where(x => x.Type == ClaimTypes.Upn).FirstOrDefault().Value;

    //do something to retrieve appropriate value for current user
    int CompanyUserID = 4000333;

    context.AuthenticationTicket.Identity.AddClaim(new Claim("CompanyUserID", CompanyUserID.ToString()));

    return Task.FromResult(0);
}
```

Again, a good candidate for these claims are broad, static bits of information that will not change, or at least the frequency of change should be much longer than the typical claims token expiration period (as the new value would not be received until next user login).
