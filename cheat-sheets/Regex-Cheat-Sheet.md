---
title: Regex Cheat Sheet
layout: page
---
# Regular Expressions Cheat Sheet

A regular expression is a sequence of characters defining a search expression, used for pattern matching in strings

## References

[Refiddle](http://refiddle.com/) - an online regex utility, similar to JSFiddle

## Characters

### Anchors

- `^`   - start of line
- `\A`  - start of string
- `$`   - End of Line
- `\Z`  - End of string
- `\b`  - Word boundary
- `\B`  - Not word boundary
- `\<`  - Start of word
- `\>`  - End of word

### Character Classes

- `\c`  - Control character
- `\s`  - White space
- `\S`  - Not white space
- `\d`  - Digit
- `\D`  - Not digit
- `\w`  - word
- `\W`  - Not word

### Quantifiers

- `*`   - 0 or more
- `*?`  - 0 or more, ungreedy
- `+`   - 1 or more
- `+?`  - 1 or more, ungreedy
- `?`   - 0 or 1
- `??`  - 0 or 1, ungreedy
- `{3}` - exactly 3
- `{3,}`    - 3 or more
- `{3,5}`   - 3, 4 or 5
- `{3,5}?`  - 3, 4 or 5, ungreedy

### Ranges

- `.`   - Any character except new line(\n)
- `(a|b)`   - a or b
- `(...)`   - group
- `(?:...)` - passive group
- `[abc]`   - Range (a or b or c)
- `[^abc]`  - Not a or b or c
- `[a-q]`   - Letter between a and q
- `[A-Q]`   - Upper case letter between A and Q
- `[0-7]`   - Digit between 0 and 7

### Pattern Modifiers

- `g`   - Global match
- `i`   - Case-insensitive
- `m`   - Multiple lines
- `s`   - Treat string as single line
- `x`   - Allow comments and whitespace in pattern
- `e`   - Evaluate replacement
- `U`   - Ungreedy pattern

### Special Characters

- `\`   - Escape character
- `\n`  - New Line
- `\r`  - Carriage return
- `\t`  - Tab
- `\v`  - Vertical tab
- `\f`  - Form feed
- `\a`  - Alarm
- `[\b] - Backspace
- `\e`  - Escape

## Examples

### Phone Number

```
/[0-9]{3}-[0-9]{3}-[0-9]{4}/
```
```
123-456-9000 - match
456-9000 - no match
1234569000 - no match
```


### Email

```
/[^\s@]+@[^\s@]+\.[^\s@]+/
```
```
brian@gmail.com - match
briangmail.com - no match
brian.vp@gmail.com - match
brian@@gmail.com - no match
```

### Date

```
/\d{1,2}\/\d{1,2}/\d{2,4}/
```
```
11/8/1980 - match
11/8/80 - match
118/80 - no match
111/8/80 - no match
```