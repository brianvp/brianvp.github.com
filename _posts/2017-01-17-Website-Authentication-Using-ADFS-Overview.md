---
title: 'Website Authentication using ADFS Overview'
date: 2017-01-17T00:00:00+00:00
author: brianvp
layout: post
permalink: /2017/01/17/Website-Authentication-Using-ADFS-Overview/
categories:
  - General
tags:
  - ADFS
  - ASP.NET Identity
---

This is the first in a series of posts going over the use of Active Directory Federation Services (AD FS).  For roughly the past year, I've been exploring authentication options for an external website project at work. Previously we focused on using Windows Authentication, but that only really works for Intranet applications, and not so well or at all on mobile / tablet environments.  Since this application is only for employees, I wanted to use existing Active Directory resources.  I briefly explored oAuth as an option, but since we would not be using google/facebook etc for credentials, setting up our own oAuth provider seemed to be as much or more work than utilizing AD FS.  Specifically my design goals were:

- Implement Single Sign-on.  Users should not have to log into multiple applications / minimize number of logins
- Use existing domain accounts - Users do not need another login to manage
- Do not allow the application to access or store user credentials / secrets - If you are going to use a common credential between websites, *no* individual website should have access to these credentials directly. At best, each site would need to re-implement security features, and at worst, one site failing to implement security properly exposes all the sites.
- Work well for Internet and Intranet applications.  While Windows Authentication worked for us in the past, it doesn't work well in a post-windows pc world.

Early last year, I created two demo projects, one using oAuth, and the other using AD FS.  Both were successful, but the oAuth solution is using google, so after reviewing with my other developers we decided to use AD FS.  At present, we have two live applications using AD FS, and so far has worked out really well.  Below are the demo projects for reference:

- [oAuth Demo Project](https://github.com/brianvp/authentication-examples/tree/master/ModelManagerOAuthIndividual)
- [AD FS Demo Project](https://github.com/brianvp/authentication-examples/tree/master/ModelManagerADFS)

The rest of this post describes key points of the AD FS architecture 

## AD FS Overview

Active Directory Federation Services is a service that allows sharing identity information between "trusted" partners, called a "federation". At a high level, it allows a website to delegate authentication to a trusted service, and accept a "claim" from this service on the user's behalf to make authorization decisions.  For example:

- John Doe wants to access the corporate payroll site
- The payroll site requires users to login in (obviously)
- When John hits the payroll site, he is not authenticated, so the payroll site redirects John to the AD FS login page
- John enters his credentials (user name and password) to the AD FS login site.  
- The AD FS site verifies the credentials - if valid, it generates a "claims token", containing certain information about John. This typically includes his username (johnd), full name, "John Doe", email, "john.doe@company.com", and any other property the AD FS service is configured to send. 
- AD FS redirects the user back to the payroll site, sending along the claims token
- The payroll site now assumes that the original user *is* John Doe, and presents resources John Doe is authorized to use.  

Additionally, the claims token is typically stored in a cookie, so if John closes his browser and reloads the payroll site, it can use the existing claims token without requring John to re-log in to AD FS.  

## Terminology

Some key terminology:

- Federation - A group of two or more independant sites / services, that are in a special trust relationship.  With the proper configuration, theoretically any service could be added to the exising AD Federation.  Much of the work required to set up AD FS is managaging this federation, both on the AD FS service, and the website.  
- Account Federation Server - Issues the claims tokens to users based on authenticating that user, basically the AD FS server
- Relying Party - the website using the claim for authenticating & authorizing the user
- Single Sign-on / SSO - a user authentication mechanism that allows a user to use one login to access multiple applications
- Claim - A statement about the user.  There are three types of claims - Identity Claims, Group Claims, and Custom Claims
    - An identity claim is basic information about the user e.g. username
    - A group claim is information about which domain groups the user belongs to e.g. PayrollUsers
    - a custom claim is a key/value pair that can be used by applications.  For example, we use a corporate user identity code that is standard between all web applications.  

## Components

Below is a simplified view of what my current AD FS system looks like

![ADFS Architecture](/assets/adfs_architecture.png)

- The AD FS Federation server is a Windows 2012 R2 Instance responsible for:
    - Registering each Relying Party / Web Server / Website
    - defining what claims metdata is in in the claims token
    - Provides a login endpoint for users to enter credentials into
    - Provides a signout endpoint to clear the claims token
    - Initially we used a 2008 R2 Instance, but upgraded to a 2012 R2 instance.  The configurations are largely the same, but one key improvement is the ability customize the user login page (the 2008 R2 page looks terrible on mobile devices)
- The Web Servers are Windows Server (2008 R2 - 2012 R2) instances which host one or more websites configured to used AD FS
    - The websites are built with ASP.NET / MVC / Web API
    - use Owin middleware for communicating with AD FS
        - redirecting the user to the AD FS Server to login
        - redirecting the user to the AD FS Server to sign out
        - Processing the Claims Token / making the claim available to the web application
    - contain AD FS endpoint / configuration metadata
- The User Instances (PCs / tablets / mobile) access the websites
    - Upon hitting the Web Server instance, user is redirected to the AD FS Server
    - Upon successful login, the claims token is stored in a cookie in the user's browser.
    - The cookie is read by the website after the AD FS Server redirects the user back to the website.  One of the most important configurations in AD FS is specifing the proper redirect endpoints.  
    - The website will continue to use the browser cookie until the user signs out, or the cookie expires.   
    - Currently we have a cookie expiration of 24 hours, which will force the user to log in at most once per day.

## Resources

- [Microsoft Overview](https://msdn.microsoft.com/en-us/library/bb897402.aspx)
- [Excellent ServerFault Overview](http://serverfault.com/questions/708669/what-is-adfs-active-directory-federation-services)
- [Detailed AD FS Terminology](https://technet.microsoft.com/en-us/library/cc754236(v=ws.11).aspx)
