---
title: 'Converting JSON Types'
date: 2016-10-17T00:00:00+00:00
author: brianvp
layout: post
permalink: /2016/10/17/Converting-JSON-Types/
categories:
  - General
tags:
  - csv
  - JSON
  - Powershell
---

In a [post](/2015/10/08/generating-json-from-csv-using-powershell/ ) last year, I found a simple csv to JSON technique that worked well - but today I noticed a significant limitation - all numbers were converted to strings, which isn't very useful for setting up mock data for `TypeScript`.

I created the simple `node.js` file below to convert an existing JSON file (using the technique above), by specifying one or more values to convert in the file.  It's not especially robust, but it works for my current needs.  A few comments:

- input file must be `utf-8`
- null values converted to string type get set to blank values.
- strings of `"null"` are converted to `null` for int or float types.
- you may specify any number of inputs fields.  
- Only supports simple file structures, no nested arrays inside the JSON objects.

### gist

<script src="https://gist.github.com/brianvp/3fdc794b6cc4330fad5c3705dfc5537d.js"></script>