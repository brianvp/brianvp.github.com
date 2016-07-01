---
title: Markdown Cheat Sheet
layout: page
---
# Markdown Cheatsheet

Markdown is a plain-text formatting syntax that is converted to HTML using a Markdown processor.   It was originally created by John Gruber, 
and has evolved into a number of different "flavors".  As such, not all markdown syntax is universal and supported.  Most markdown files use the `*.md` extension

### Markdown variants

#### GFM - Github Flavored Markdown
* adds support for tables
* Syntax highlighting
* treats newlines as real line breaks

#### CommonMark
* presents a tighter specification of Markdown syntax


## References
---

* [en.wikipedia.org/wiki/Markdown](https://en.wikipedia.org/wiki/Markdown)
* [daringfireball.net/projects/markdown/](https://daringfireball.net/projects/markdown/) - John Gruber - Original Author of Markdown
* [github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) - Inspired most of this cheat sheat

## Headers
---

```
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

# H1
## H2
### H3
#### H4
##### H5
###### H6

## Emphasis
---

```
*italics* _italics_

**bold** __bold__

**_bolditalic_**

~~strikethrough~~
```

*italics* _italics_

**bold** __bold__

**_bolditalic_**

~~strikethrough~~

## Lists
---

```
### ordered
1. One
2. two
3. three


### unordered
* one
* two
* three


- one
- two
- three


+ one
+ two
+ three


### sub-lists (doesn't appear to be working)
1. One
2. two
..* two-point-five 
3. three
..1. next
```

### ordered
1. One
2. two
3. three


### unordered
* one
* two
* three


- one
- two
- three


+ one
+ two
+ three


### sub-lists (doesn't appear to be working)
1. One
2. two
..* two-point-five 
3. three
..1. next


## Links
---

```
[Inline Link](https://www.google.com)

[Inline Link with toolltip](https://www.google.com "Google.com")

```

[Inline Link](https://www.google.com)

[Inline Link with toolltip](https://www.google.com "Google.com")

## Images
---

```
![alt text](https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png)
```

![alt text](https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png)


## Code & Syntax Highlighting
---

Inline code `System.Net.Serialization` example - use backticks around code


### Code Blocks

use 3x Backticks around code block.   Some markdown renders support syntax highlighting, so you can include the language behind the backticks

```
--- Javascript (replace dash with back-tick)
 // code
---

```

```Javascript
var s = "Hello, World!";
alert(s);
```

```Python
s = "Hello, World!"
print s
```

```
An unspecified Lanuage
```

## Tables
---

Tables are not part of the core markdown spec, some renders support the (GFM)

```
|name|age|city|
|Bob|35|Orange City|
|Susan|46|Sioux Center| 

```

|name|age|city|
|Bob|35|Orange City|
|Susan|46|Sioux Center| 

##Blockquotes
---

```
>line 1
>Line 2



>I am the very model of a modern Major-General,
I've information vegetable, animal, and mineral,
I know the kings of England, and I quote the fights historical
From Marathon to Waterloo, in order categorical;
I'm very well acquainted, too, with matters mathematical,
I understand equations, both the simple and quadratical,
About binomial theorem I'm teeming with a lot o' news, 
With many cheerful facts about the square of the hypotenuse.
I'm very good at integral and differential calculus;
I know the scientific names of beings animalculous:
In short, in matters vegetable, animal, and mineral,
I am the very model of a modern Major-General
```

>line 1
>Line 2

Longer Text

>I am the very model of a modern Major-General,
I've information vegetable, animal, and mineral,
I know the kings of England, and I quote the fights historical
From Marathon to Waterloo, in order categorical;
I'm very well acquainted, too, with matters mathematical,
I understand equations, both the simple and quadratical,
About binomial theorem I'm teeming with a lot o' news, 
With many cheerful facts about the square of the hypotenuse.
I'm very good at integral and differential calculus;
I know the scientific names of beings animalculous:
In short, in matters vegetable, animal, and mineral,
I am the very model of a modern Major-General

## Inline HTML
---

```
<table>
    <tr><td>One</td></tr>
    <tr><td>Two</td></tr>
    <tr><td>Three</td></tr>
</table>
```

<table>
    <tr><td>One</td></tr>
    <tr><td>Two</td></tr>
    <tr><td>Three</td></tr>
</table>

## Horzizontal Rule
---

```
---

***

___
```


---

***

___

## Line Breaks
---
```
Paragraph

Same paragraph (one line break)


different paragraph (two lines breaks)
```

Paragraph

Same paragraph (one line break)


different paragraph (two lines breaks)



## Revision History
--- 

* 5/11/2016 - Initial Version