---
url: https://medium.com/notes-and-tips-in-full-stack-development/modify-binary-files-with-vim-c35b40c499e
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/modify-binary-files-with-vim-c35b40c499e
title: Modify binary files with VIM
subtitle: Edit binary files in Linux with Vim
slug: modify-binary-files-with-vim
description: Sometime I need to open binaries, change few bytes with own values and
  execute again. Found one of the most easy solution, that have on each machine.
tags:
- vim
- debug
- linux
- cracking
- hacking
author: Michael Nikitochkin
username: miry
---

# Modify binary files with VIM

> TL;DR `xxd ./bin/app | vim —` and `:%!xxd -r > ./bin/new_app`

![Photo by Markus Spiske on Unsplash](/assets/2019-01-13-modify-binary-files-with-vim-0_FbFs8aNmqNLKw4BM.jpeg)

When I was a student, I used a lot Windows. It was hard times and we had to crack tools like [Total Commander](https://www.ghisler.com/) or [FAR manager](https://www.farmanager.com/) or upgrade Diablo’s heroes in saves files. I used editors like: [WinHex](http://www.winhex.com/winhex/hex-editor.html) and [UltraEdit](https://www.ultraedit.com/) .

Today I came to similar problem modify binary file in Linux terminal. I started search with one requirements:

1. it should be cross distributive solution

1. should be easy to use with Vim (as main my editor for linux machines)

Few searches and I came to: [**xxd**](http://vim.wikia.com/wiki/Hex_dump). It is a part of `vim-common` package, that lucky would be installed on each system. I need to learn only how to read, modify and write.

**Read**: Use `xxd` to decode binary and redirect output to `vim`: `xxd ./bin/app | vim -`

**Modification**: You can edit evrything - because it is just a text. There are 3 big sections. First column is the addresses, second is the hex representation of a binary file, and third one is the ASCII preview of the binary. With this solution preview would not be updated on changes.

**Write**: After you modify required bytes in the middle section. Instead of type `:w` , you should run `xdd`: `:%!xdd -r > ./bin/new_app` .

# Summary

[![Youtube](https://img.youtube.com/vi/30xiI21RraQ/hqdefault.jpg)](https://www.youtube.com/watch?v=30xiI21RraQ)

![That’s all Folks](/assets/2019-01-13-modify-binary-files-with-vim-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


