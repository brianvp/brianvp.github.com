---
title: 'ADFS Authentication - Adding to Existing Site'
date: 2017-01-27T00:00:00+00:00
author: brianvp
layout: post
permalink: /2017/01/27/ADFS-Authentication-Adding-to-Existing-Site/
categories:
  - General
tags:
  - ADFS
  - ASP.NET Identity
  - Owin
---

The Visual Studio new project template is a handy wizard for setting up AD FS on a new site, but the wizard isn't available for existing sites.  Additionally, the wizard forces you to include ASP.NET MVC components, even if you just need to set up a Web API back-end.  This post shows you how to do this manually. 

1. Make sure SSL is enabled in your project.

2. Set The SSL URL to the one provided by your AD FS administrator, or configure AD FS to use your website url/port. e.g. `https://localhost:44300/`

3. Add required packages (search for these on NuGet)
    ![manual adfs nuget](/assets/manual_adfs_nuget.png)
    - `Microsoft.Owin`
    - `Microsoft.Owin.Security`
    - `Microsoft.Owin.Security.Cookies`
    - `Microsoft.Owin.Security.WsFederation`
    - `Microsoft.Owin.Host.SystemWeb`
    
4. Add AD FS Metadata to `Web.config`

```xml
<appSettings>
    <add key="ida:ADFSMetadata" value="https://example.com/federationmetadata/2007-06/federationmetadata.xml" />
    <add key="ida:Wtrealm" value="https://localhost:44300/" />
</appSettings>
```

5. Add a `Startup.cs` file/class if it does not already exist

6. Add configuration hooks to `Startup.cs`. Note that if you have not added `Microsoft.Owin.Host.SystemWeb` to your packages `Startup.Configuration()` will not fire.

```csharp
public void Configuration(IAppBuilder app)
{
    ConfigureAuth(app);
}

private void ConfigureAuth(IAppBuilder app)
{
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);
    app.UseCookieAuthentication(new CookieAuthenticationOptions());
    app.UseWsFederationAuthentication(
        new WsFederationAuthenticationOptions
        {
            Wtrealm = ConfigurationManager.AppSettings["ida:Wtrealm"],
            MetadataAddress = ConfigurationManager.AppSettings["ida:ADFSMetadata"]
        });
}
           
```

7. Add Authorization to controllers / routes.  
    - Method 1: Set global filter

        ```csharp
        public static void Register(HttpConfiguration config)
        {
            ...

            // Authorize all api routes
            config.Filters.Add(new AuthorizeAttribute());

            ...
        }
        ```

    - Method 2: Add Authorize Attribute to controller

        ```csharp
        [Authorize]
        public class PersonController : ApiController
        {
            ...
        }
        ```

8.  Build and Run your app.  Your routes requiring authorization should now redirect you to the AD FS login page

![manual adfs signin](/assets/manual_adfs_signin.png)

