---
url: https://medium.com/notes-and-tips-in-full-stack-development/crystal-string-and-heredoc-with-escaped-characters-for-json-4c3e00fd7a65
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/crystal-string-and-heredoc-with-escaped-characters-for-json-4c3e00fd7a65
title: Crystal string and heredoc with escaped characters for JSON
subtitle: 'TLDR: Clarify a problem why some JSON content could be not parsable via
  directives and how to fix it'
slug: crystal-string-and-heredoc-with-escaped-characters-for-json
description: ""
tags:
- crystal-lang
- json
author: Michael Nikitochkin
username: miry
---

# Crystal string and heredoc with escaped characters for JSON

### TLDR: Clarify a problem why some JSON content could be not parsable via directives and how to fix it

```
require "json"
```

```
pp JSON.parse(%|{"example1": "string without new line"}|)
# > {"example1" => "string without new line"}
```

```
pp JSON.parse <<-CONTENT
{
  "example2": "heredoc without new line"
}
CONTENT
# > {"example2" => "heredoc without new line"}
```

```
pp JSON.parse(%|{"example3": "string with new line\n"}|)
# > Unhandled exception: Unexpected char '
#   ' at line 1, column 35 (JSON::ParseException)
```

```
pp JSON.parse <<-CONTENT
{
  "example4": "heredoc with new line\n"
}
CONTENT
# > Unhandled exception: Unexpected char '
#   ' at line 1, column 35 (JSON::ParseException)
```

Review the command: `JSON.parse(%|{“foo”: “string with new line\n”}|)` . It produces prase error:

```
Unhandled exception: Unexpected char '
' at line 1, column 35 (JSON::ParseException)
```

The error printed hidden character as new line. It gave a hint, that something is not escaped. It looks like, JSON does allow to close a string on next lines:

```
$ echo "{\"foo\": \"string with new line\n\"}" | jq .
parse error: Invalid string: control characters from U+0000 through U+001F must be escaped at line 2, column 1
```

### How to fix the problem

A heredoc and string generally allows interpolation and escapes.

Interpolation can be disabled by using a non-interpolating string literal like `%q()` or escaping backslash `\\` .

The opening heredoc identifier is enclosed in single quotes.

```
pp JSON.parse(%q("example3": "string with new line\n"}))
# > {"example3" => "string with new line\n"}
```

```
pp JSON.parse <<-'CONTENT'
{
  "example4": "heredoc with new line\n"
}
CONTENT
# > {"example4" => "heredoc with new line\n"}
```

# References

https://crystal-lang.org/reference/1.4/syntax_and_semantics/literals/string.html#heredoc

https://www.json.org/json-en.html


