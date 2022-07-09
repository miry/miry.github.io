---
url: https://medium.com/notes-and-tips-in-full-stack-development/simple-dynamic-template-687ffbefce71
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/simple-dynamic-template-687ffbefce71
title: Simple Dynamic Template
subtitle: Sometimes we need to define a dynamic template before defining variables
  used in the template. Let me show a few examples how to solve this…
slug: simple-dynamic-template
description: ""
tags:
- ruby
author: Michael Nikitochkin
username: miry
---

# Simple Dynamic Template

Sometimes we need to define a dynamic template before defining variables used in the template. Let me show a few examples how to solve this problem in Ruby.

The best way is to use the [format string features from the Ruby](http://ruby-doc.org/core-2.0.0/String.html#method-i-25):

```
string = "echo %{expr}" # -> "echo %{expr}"
string % {expr: 1}      # -> "echo 1"
```
> *[simple-template-format.rb view raw](https://gist.githubusercontent.com/marchi-martius/85e35d4ee2cf3551679437797192437e/raw/2092ec1d91138f15debdff539a4c9ce03586c3e1/simple-template-format.rb)*

The first method is based on the ability to evaluate the Ruby expression in string:

```
string = 'echo #{expr}'  # -> "echo \#{expr}"
expr = 1                 # -> 1
eval("\"#{string}\"")    # -> "echo 1"
```
> *[simple-template-eval.rb view raw](https://gist.githubusercontent.com/marchi-martius/974af47d295414031c129ecdfa67558b/raw/11d9f27c115077c44f7eedee0b83ce4899fea7b1/simple-template-eval.rb)*


