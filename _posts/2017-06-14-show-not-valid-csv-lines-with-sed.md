---
url: https://medium.com/notes-and-tips-in-full-stack-development/show-not-valid-csv-lines-with-sed-fa82da01f133
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/show-not-valid-csv-lines-with-sed-fa82da01f133
title: Show Not Valid CSV Lines with Sed
subtitle: I have an issue with invalid formatted CSV file. First step show lines with
  invalid lines.
slug: show-not-valid-csv-lines-with-sed
description: ""
tags:
- data-science
author: Michael Nikitochkin
username: miry
---

![](/assets/2017-06-14-show-not-valid-csv-lines-with-sed-1_fOVQOYA8QGQHJ5MaaCg3sw.jpeg)

# Show Not Valid CSV Lines with Sed

I have an issue with invalid formatted CSV file. First step show lines with invalid lines.

```
$ sed -n '/"[^",]*"[^",]*"[^",]*",/,1p' <fileName>
```

Then I googled a way to replace symbol inside quotes. And I read the next manual [http://sed.sourceforge.net/sed1line.txt.](http://sed.sourceforge.net/sed1line.txt.) So I created a sed script with the next content, called it **script.sed**:

```
s/\",\"/\$XXXX\$/g;
:a
s/\([^,]\)"\([^,]\)/\1'\2/g
ta
s/\$XXXX\$/\",\"/g;

```
> *[invalid-csv.sed view raw](https://gist.githubusercontent.com/marchi-martius/99cd3e4e1a7d436854a73d845583c49e/raw/bc05e1250730729308f544d835fdd07550cda291/invalid-csv.sed)*

Next we just do:

```
$ sed -f script.sed <fileName>
```

And we get a normal csv format file in the output. Next we just add the argument to apply that in this file.

```
$ sed -i .bak -f script.sed <fileName>
```


