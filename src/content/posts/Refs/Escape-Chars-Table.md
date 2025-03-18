---
title: Escape Chars Table
description: This is the first post of my new Astro blog.
published: 2024-01-18 17:37:48
tags:
- Table
category: Reference
---

The full table of escape characters.

<!--more-->


| Escape Character | Description                                                                             | Unicode Value |
| ---------------- | --------------------------------------------------------------------------------------- | ------------- |
| `\\`             | Backslash                                                                               | \u005C        |
| `\'`             | Single quote                                                                            | \u0027        |
| `\"`             | Double quote                                                                            | \u0022        |
| `\?`             | Question mark                                                                           | \u003F        |
| `\a`             | Alert (bell)                                                                            | \u0007        |
| `\b`             | Backspace                                                                               | \u0008        |
| `\f`             | Formfeed                                                                                | \u000C        |
| `\n`             | Newline (line feed)                                                                     | \u000A        |
| `\r`             | Carriage return                                                                         | \u000D        |
| `\t`             | Horizontal tab                                                                          | \u0009        |
| `\v`             | Vertical tab                                                                            | \u000B        |
| `\0`             | Null character                                                                          | \u0000        |
| `\ooo`           | Octal number of one to three digits (where `ooo` is the octal number)                   |
| `\xhh`           | Hexadecimal number of one or more digits (where `hh` is the hexadecimal number)         |
| `\uhhhh`         | Unicode character (16-bit) in hexadecimal notation (where `hhhh` is the hex number)     |
| `\Uhhhhhhhh`     | Unicode character (32-bit) in hexadecimal notation (where `hhhhhhhh` is the hex number) |
| `\N{name}`       | Unicode character named `name` in the Unicode database (not supported in all languages) |
