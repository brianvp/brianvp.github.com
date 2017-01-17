---
title: 'Understanding Relative Urls'
date: 2017-01-16T00:00:00+00:00
author: brianvp
layout: post
permalink: /2017/01/16/understanding-relative-urls/
categories:
  - General
tags:
  - html
---

Something you use all the time, but don't always realize, is how a web browser will request resources in html files with relative url's.  

for example:

```
    <script src="libs/jquery.min.js" type="text/javascript"></script>
```

will be served from example.com as `http://example.com/libs/jquery.min.js` - provided the website is sitting in the web server root e.g `wwwroot` or `/var/www/html`.  

If, instead, the app is sitting in a sub-folder / virtual directory, the above code will still work, even thought it may now come from `http://example.com/salesapp/libs/jquery.min.js/`.  This is great, as it lets us decouple our application structure from where it is deployed.  

What if we changed the URL a bit?  say to:

```
    <script src="/libs/jquery.min.js" type="text/javascript"></script>
```

With this change, the leading `/` indicates the root of the website.  It is still a relative path, but it is "relative" to the root of the website.  `http://example.com/libs/jquery.min.js` will still work, but `http://example.com/salesapp/libs/jquery.min.js` will not.  This can be a subtle error, as most applications run locally are sitting at `http://localhost:12345` or whatever, but many are not deployed to the web root.  

Using the leading slash removes a lot of future flexibility.   So why would you want it?  Well, consider this scenario.  You have a script file that has a jquery dependency, which lives at http://example.com/salesapp/scripts/accounts.js.  This reference will no longer work:

```
    <script src="libs/jquery.min.js" type="text/javascript"></script>
```

It will return as `http://example.com/salesapp/scripts/libs/jquery.min.js` oops.  

The reason this doesn't work is that the relative path is build based on your current window location, which will change with different url's:

```
http://example.com
http://example.com/salesapp
http://example.com/salesapp/accounts
```

It will have to be one of these:

```
    <script src="/libs/jquery.min.js" type="text/javascript"></script>
    <script src="../libs/jquery.min.js" type="text/javascript"></script>
    <script src="http://example.com/libs/jquery.min.js" type="text/javascript"></script>
```

Out of the three, the 2nd is probably the most flexible, but is cumbersome to read and write (especially if your source trees are deep).

What you really want to do is set your url relative to the path of the application.  This can be  using the `<base>` tag.  According to [mdn](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base)

"The HTML <base> element specifies the base URL to use for all relative URLs contained within a document."

For example, if you set up the `<base>` tag as follows:

```
<head>
    <base href="/">
</head>
```

This effecively sets all relative paths relative to the root of the website
`libs/jquery.min.js` will be read as `/libs/jquery.min.js`, and thus `http://example.com/libs/jquery.min.js/`

To set an application specific path, you could use:

```
<head>
    <base href="/salesapp/">
</head>
```

`libs/jquery.min.js` will be read as `/salesapp/libs/jquery.min.js`, and thus `http://example.com/salesapp/libs/jquery.min.js/`

this works well when deploying to different endpoints as only the `<base>` reference need to change, which can be easily handled via a build system.  

Be careful though, specifing relative paths with the leading slash will *ignore* the specified `<base>`.

Lastly, you can also specify an empty/current directory base path

```
<head>
    <base href="">
</head>
```

or

```
<head>
    <base href="./">
</head>
```

Which is effectively the same as specifying no `<base>`.  You might use this for local development.

Addtionally, you can dynamically write the `<base>` with a script, but doesnt appear to be a [recommended](http://stackoverflow.com/questions/3494954/how-do-i-set-a-pages-base-href-in-javascript) approach. It seems better to leave this up to your build / depolyment system.  

The good news is that broken paths are generally easy to detect when your browser is in development mode - just look for the red requests in the network tab, and check the path.  You'll know something is off when the path is partially correct, but contains additional levels or missing levels alltogether.   


