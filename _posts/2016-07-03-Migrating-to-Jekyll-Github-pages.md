---
title: Migrating to Jekyll Github Pages
date: 2016-07-03T02:18:54+00:00
author: brianvp
layout: post
permalink: /2016/07/03/Migrating-to-Jekyll-Github-pages/
categories:
  - General
---

As of yesterday my blog is being hosted by [GitHub Pages](https://pages.github.com/), using the [Jekyll](https://jekyllrb.com/) static page engine.  I've been using Wordpress hosted on a [DigitalOcean](https://www.digitalocean.com/) droplet for the past year, and I haven't been happy with the platform.  Stability was a major concern, and the blog authoring didn't suit technical blogging very well. And finally, Github Pages + Jekyll has some seriously cool features.

## Wordpress Stability

Over nine months, my site was down for at least *twice* a week. Not cool. As best as I can tell, my instance was running out of memory, which is ridiculous for a low traffic site...

```
Error Establishing Database Connection
```

ugh.


Initially I was very exicted about using WordPress.  DigitialOcean uses the concept of "droplets", that can be loaded with a number of Linux server configurations.  I chose the $10/mo option, which gave me 1GB memory, 1 processor core, and a 30GB SSD.  Surely enough for a personal blog?  Unfortunately, within a month or two of running the new site, it started going down, frequently.  

The first thing I noticed was a large number of "xmlrpc" attacks

![XML rPC Attack](/assets/xml-rpc-attack.PNG)

The XML rpc service is used to upload posts to wordpress from clients.  As best as I can gather, these requests were flooding the server to the point that memory utilization was over 99%, taking out MySQL (not really sure why).  My initial response was to enable the firewall `ufw`, and add each attacker to the list.  I did consider disabling the xmlrpc service, but I didn't feel great about removing a feature that I may want to use in the future.  

Manually updating the firewall wasn't very effective however, as the IP's frequently change, and I have no warning of when it starts happening.  So my next step was to install `fail2ban`.  I configured fail2ban to block attackers after 3 failed attempts of calling the xmlrpc service.  This was a much better approach, as I didn't need to monitor the site, and my IP banned list wouldn't grow very long (fail2ban reverses the ban after a period of time).  But that *still* didn't work.  I mean, it was better, but my site was still going offline.  

I considered adding more memory to the instance, but I felt that paying another $10 / mo for 2GB of memory was too much for a personal site.  I learned that the swap file is disabled in DigitalOcean's linux configurations, so I enabled it with 4GB.  This helped some, but my site was still going offline at least once a week - and the swap file wasn't even full! 

At this point I was simply fed up with the platform, and not sure what to do next.  I set up the blog for blogging, not dealing with attackers / server performance issues!  
 
## Wordpress Editing 

I was also getting fed up with writing technical articles in Wordpress.  My initial Idea was simple:

1. Research Article, taking notes in Google Docs, creating code samples, etc.  
2. Outline the article in Google Docs
3. Write draft of article in Google Docs
4. Upload the draft into Wordpress and update the final draft.

This didn't work as seamlessly as I would have liked.  First, there is no way to directly upload from Google Docs.  There used to be a feature for this, but it was removed a few years ago.  So I ended up doing a lot of copying and pasting.  For the plain text content this wasn't too bad, but it was a small nightmare for code.  In addition to frequent indentation issues, the wordpress editor frequently rewrote code  like this: `List<string>` into: `List&lt;string%gt;` not cool.  

## Jekyll 

A few months ago I ran into a blog that used Jekyll + GitHub Pages.  The initial feature that really turned me on was that the entire blog was hosted in a [GitHub repo](https://github.com/brianvp/brianvp.github.com)!  Editing is done in [MarkDown](https://daringfireball.net/projects/markdown/), in your editor of choice [Visual Studio Code](https://code.visualstudio.com/) is a great choice for this as it has a built-in MarkDown preview mode.  

![Visual Studio Code Editing](/assets/Visual-Studio-Code-Editing.PNG)


Compared to WordPress, the killer feature of Jekyll is that there is *no* database!  The site contents are all compiled into HTML during deployment, so the server only needs to serve HTML files - no server side rending per page request!  The other benefit to this is that GitHub can apparently host these sites very cheaply, because GitHub Pages is free!  And in the event I need more that what GitHub provides, I can deploy a Jekyll instance with most cloud providers (including Digital Ocean).  

The only problem with Jekyll is that it isn't extremly easy to set up a site.  There are existing sites / layouts available, but they are nowhere near as easy to set up as say a WordPress theme.  This is a small tradeoff though, as once the site is up and running you shouldn't need to touch many configuration details.  All told, it took me about 10-15 hours over the last two months to migrate my site.  

An invaluable tool was Ben Balter's [Wordpress-to-Jekyll-Exporter](https://github.com/benbalter/wordpress-to-jekyll-exporter). This easily converted all my WordPress posts to MarkDown files, and even kept the correct image links.  The code snippets were a bit off, but that wasn't a big deal to fix.  It doesn't fully convert to Markdown, as my posts still had several `<spans>`'s in the content, but this was way better than rewriting every single post...

### List of Resources for Setting up a Jekyll Blog

- [Github Pages](https://pages.github.com/)
- [Main Jekyll Site](https://jekyllrb.com/)
- [Jekyll Bootstrap](https://github.com/plusjade/jekyll-bootstrap/) - quickly get a blog engine up and running
- [Jekyll Themes](http://jekyllthemes.org/ ) 
- [Favicon Generator](http://www.favicomatic.com/) - super easy site, works well converting images

## Conclusion

I really wanted to like WordPress, and I do think it has it's place.  But for a small developer site, it really isn't worth the effort needed to keep it running.  Granted I could use wordpress.com as my host, but then I still have authoring issues. 

On a postitive side, I learned a lot about Linux administration.   It was actually kind of fun for a while - I'd see my site down, restart, and it would be down again within minutes (at the worst point).  This involved a furious scramble trying to figure out WTH was going on - and lots of on-the-fly learning.

 It's also a good reminder for developers in general that you should assume that *anything* you put out there *will* be attacked.  Of course, using a popular package with well-known attack surfaces doesn't help, but you should always assume that some malicious person/bot is going to find your site, and *will* try to harm it.  So you need to be prepared, and leaving things up to your friendly admin / ops staff is not a good strategy.