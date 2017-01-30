---
title: 'Generate Queryable List of Names from Excel'
date: 2017-01-30T00:00:00+00:00
author: brianvp
layout: post
permalink: /2017/01/30/generate-queryable-list-of-names-from-excel/
categories:
  - tips
tags:
  - excel
  - sql
---

Here's a handy excel formula to remember:

```
=CONCATENATE(CHAR(39),A2, " ", B2, CHAR(39), CHAR(44))
```

![list of names excel](/assets/list_of_names_excel.png)

For quickly generating a list of queryable list of names:

```sql
Select * from dbo.Person
where Name in
(
'Bill Gates',
'Steve Jobs',
'Larry Ellison',
'Steve Case',
'Jon Carmack',
'Elon Musk',
'Sergey Brin'
)
```

This can be handy if you get spreadsheets from people and want to quickly filter results in the database.   I did this today for someone who wanted a list of user names for about 40 people, far too many to spend half an hour typing these in!  