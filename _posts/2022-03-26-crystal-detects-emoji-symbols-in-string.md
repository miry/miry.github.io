---
url: https://medium.com/notes-and-tips-in-full-stack-development/crystal-detects-emoji-symbols-in-string-b6759ab94625
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/crystal-detects-emoji-symbols-in-string-b6759ab94625
title: Crystal detects Emoji symbols in String
subtitle: Problem is to identify unicode characters that has different byte size,
  but single symbol in render. Such of the symbols Emoji.
slug: crystal-detects-emoji-symbols-in-string
description: ""
tags:
- crystal-lang
- emoji
author: Michael Nikitochkin
username: miry
---

# Crystal detects Emoji symbols in String

Problem is to identify unicode characters that has different byte size, but single symbol in render. Such of the symbols Emoji.

```
subject = "🇺🇦 Ukraine"
puts subject.size
> 10
```

```
puts subject.bytesize
> 16
```

```
puts subject.chars
> ['🇺', '🇦', ' ', 'U', 'k', 'r', 'a', 'i', 'n', 'e']
```

To help developers to skip building a big *Regexp*[²](https://www.unicode.org/reports/tr51/#EBNF_and_Regex) to detect characters, introduced *String::Grapheme*[¹](https://crystal-lang.org/api/1.3.2/String/Grapheme.html).

```
puts subject.grapheme_size
> 9
```

```
puts subject.graphemes
> [String::Grapheme("🇺🇦"), String::Grapheme(' '), String::Grapheme('U'), String::Grapheme('k'), String::Grapheme('r'), String::Grapheme('a'), String::Grapheme('i'), String::Grapheme('n'), String::Grapheme('e')]
```

The result shows exactly the number of symbols to be rendered.

Example how *Grapheme* could be used. Here is original code:

```
result = ""
subject.each_char_with_index do |c, index|
  result += "<" if index == 2
  result += c
  result += ">" if index == 8
end
puts result
> 🇺🇦< Ukrain>e
```

and it converted to something very similar

```
index = 0
result = ""
subject.each_grapheme do |symbol|
  result += "<" if index == 2
  result += symbol.to_s
  result += ">" if index == 8
  index += 1
end
puts result
> 🇺🇦 <Ukraine>
```

### References

https://crystal-lang.org/api/1.3.2/String/Grapheme.html


