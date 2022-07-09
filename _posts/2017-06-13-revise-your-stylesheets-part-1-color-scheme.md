---
url: https://jtway.co/revise-your-stylesheets-part-1-color-scheme-6125f869453b
canonical_url: https://jtway.co/revise-your-stylesheets-part-1-color-scheme-6125f869453b
title: Revise Your Stylesheets. Part 1. Color Scheme
subtitle: Ever wonder how many colors exist in your project? Do you have strict style
  guides kindly prepared by the design lead? How many PSDs did…
slug: revise-your-stylesheets-part-1-color-scheme
description: ""
tags:
- web-development
- css
author: Michael Nikitochkin
username: miry
---

![](/assets/2017-06-13-revise-your-stylesheets-part-1-color-scheme-1_aa1T1Wk_GiYrOvm-oL-TUA.jpeg)

# Revise Your Stylesheets. Part 1. Color Scheme

Ever wonder how many colors exist in your project? Do you have strict style guides kindly prepared by the design lead? How many PSDs did you slice for this particular project? Do you work alone or do you have a large front-end team and no codestyle manifest? I bet that one day you’ll wake up and see a rainbow full of hex, RGB and RGBA colors spread all over the 100 SCSS files. And do you want to know how to merge 300 colors into 30? In this article you’ll see how to gather and visualize colors presence in your project.

# Foreword

Working on a big project is always challenging but sometimes it leads to a stylesheet mess especially when it comes with few redesign iterations and has more than 1 developers working on it.

In our project, we have stylesheets counting hundreds of files. It is obvious that searching for all colors from these files manually would take too much time. In this article, I will show you how to gather all colors information from your application and represent them for reviewing with jump-to-file action.

# Toolset

A few words about tools. I will use NodeJS v5.9 and es6 features such as template string, map datatype and destructuring assignment. [Xray-rails](https://github.com/brentd/xray-rails) gem for dispatching get requests to your selected editor or IDE.

# Gather information

The first thing I’m going to do is to search for files in the particular directory and gather all colors and with hex and RGB format. Here is the regexp pattern used for test hex and RGB(A) color formats: [http://regexr.com/3d3jh](http://regexr.com/3d3jh) . This also matches ID selectors with the length of 3 or 6 symbols named as color code, like this `#f1deaa`, but it’s not a big deal.

The following script reads the particular directory and checks its content recursively. If it found a file it reads its content check for regex pattern and adds to map, where a key is a color code and value is the object of place (array of files paths with line and caret position) and color index. Color index is just an integer representing a hexadecimal number. It will be used for sorting.

```
// This function process file's data line by line and test each line
// with color regex, if any color is present lets add it into color map
// with full file path and line and caret position.
function searchForColorInFile(data, filePath) {
  var result = [];
  var test;
  var lines = data.split('\n');
  for (var i = 0, len = lines.length; i < len; i++) {
    var lineStr = `${filePath}:${i+1}:`;
    while (test = colorRegexp.exec(lines[i])) {
      addToMap(test[0], lineStr + (test.index + 1), colorMap);
      result.push(test[0]);
    }
  }
  return result;
}

// Helper function that convert color to hex format,
// check if that color exist in color map,
// if exist then push new string into color data or add new color to the map.
function addToMap(color, place, map) {
  var normalizedColor = color.toLowerCase();
  var longColor = convertShortHEXtoLong(normalizedColor);
  if (map.has(longColor)) {
    var val = map.get(longColor);
    val.place.push(place);
    val.index = getColorIndex(longColor);
    map.set(longColor, val);
  } else {
    var val2 = {
      place: [],
      index: 0
    };
    val2.place.push(place);
    val2.index = getColorIndex(longColor);
    map.set(longColor, val2);
  }
  return map;
}
```
> *[stylesheets-gather.js view raw](https://gist.githubusercontent.com/marchi-martius/b90bd4348854ab3b3f07a15f50c1ec3a/raw/50c192c86315cc816c94749e7b856d28f481497b/stylesheets-gather.js)*

This code read files and directories and run methods mentioned above:

```
function processDir(path) {
  var files = [];
  if (typeof path === 'array') {
    for (var el of path) {
      files = fs.readdirSync(el);
      main(files, el);
    }
  } else {
    files = fs.readdirSync(path);
    main(files, path);
  }
}

// Main function go through a list of files
// and run color search and visualize functions.
function main(files, dir) {
  dir = dir ? dir : dirToParse;
  var data,
      filePath,
      pType,
      colors;
  for (var file of files) {
    filePath = path.resolve(dir, file);
    pType = pathType(filePath);

    if (pType === 'FILE') {
      if (filePath === path.resolve(skipFile)) continue;
      data = fs.readFileSync(filePath, 'utf-8');
      colors = searchForColorInFile(data, filePath);
      countAndPrintProcessedFiles(filePath, colors);
    } else if (pType === 'DIRECTORY') {
      processDir(filePath);
    } else {
      console.log(`${filePath} is not file`);
    }

  }
}
```
> *[stylesheets-read-dir.js view raw](https://gist.githubusercontent.com/marchi-martius/9506dc7713a7de6fbef5fc3b49d9387e/raw/467419ebc52d9d60898bd57fd189a4d5226edcf5/stylesheets-read-dir.js)*

# Visualize information

After this, I need to represent gathered information for visual reviewing. All I need here is HTML template with basic styles and iterate through all map keys and write them to file. At this step, I need a sorted array. I was using an insertion sort and color index as a sort key.

```
// Visualize function
function printToHTMLColorMap(map) {
  // Head template for generated html
  const header = `
    <!doctype html>
    <html>
    <head>
    <title>${map.size} colors (${(new Date).toLocaleDateString()})</title>
    <style>
      body {
        padding: 0;
        margin: 0;
        font-size: 0;
        display: flex;
        flex-flow: row wrap;
        justify-content: center;
        align-items: center;
        align-content: center;
        background:
          repeating-linear-gradient( 135deg,
                                     #FFF,
                                     #FFF 9px,
                                     #000 9px,
                                     #000 11px );
        background-attachment: fixed;
      }
      .color b,
      .color i {
        pointer-events: none;
      }
      .color {
        font: italic 10px/1 arial;
        color: #333;
        text-align: center;
        display: flex;
        min-width: 200px;
        height: 40px;
        text-shadow: 1px 0 0 #fff,
                    -1px 0 0 #fff,
                     0 1px 0 #fff,
                     0 -1px 0 #fff;
        flex: auto;
        flex-flow: row wrap;
        align-items: center;
        justify-content: space-around;
        align-content: center;
      }
      .color:hover {
        transform: scale(1.05);
      }
      .xray-link:visited {
        color: #777;
      }
      #modal {
        font-size: 14px;
        display: none;
        position: fixed;
        padding: 20px;
        top: 25%;
        left: 15%;
        right: 15%;
        bottom: 25%;
        background: rgba(250, 250, 250, 0.95);
        overflow: auto;
        text-shadow: 0 1px 2px #fff,
                     0 -1px 2px #fff;
      }
      #close {
        position: absolute;
        right: 5px;
        top: 2px;
      }
    </style>
    </head>
    <body>
  `;
  // Footer template for generated html,
  // containing vanilla js for modal and xray ajax links
  const footer = `
    <script>
    document.addEventListener('click', toggleModal);
    document.addEventListener('keyup', escHandler);
    function toggleModal(e) {
      console.log(e);
      var modal = document.getElementById('modal');
      var modalContent = document.getElementById('modal-content');
      var modalTitle = document.getElementById('modal-title');
      var target = e.target;
      var title = target.title;
      var bg;
      if (target.id === 'close') {
        modal.style.display = 'none';
        modal.style.background = 'rgba(250,250,250,0.95)';
      } else if (target.className === 'color') {
        bg = target.style.background;
        modalContent.innerHTML = createLinkList(title);
        modal.style.display = 'block';
        modal.style.background = bg;
        modal.style.boxShadow = '0 12px 110px ' + bg;
        modalTitle.innerText = target.getElementsByTagName('b')[0].innerText;
      } else if (target.className === 'xray-link') {
        xrayHanlder(target.text);
      } else {
        modal.style.display = 'block';
      }
    }
    function escHandler(e) {
      var modal = document.getElementById('modal');
      console.log(e);
      if (e.keyCode === 27) {
        modal.style.display = 'none';
      }
    }
    function createLinkList(text) {
      var textArr = text.split('\\n');
      var tmp = '';
      for (var i = 0, len = textArr.length; i < len; i++) {
        tmp += '<a href="#' + textArr[i]
            + '" class="xray-link">' + textArr[i] + '</a><br>';
      }
      return tmp;
    }
    function xrayHanlder(path) {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', '${xrayUrl}' + path, false);
      xhr.send();
      if (xhr.status != 200) {
        console.error(xhr.status, xhr.statusText);
      } else {
        console.log(xhr.responseText);
      }
    }
    </script>
    <div id="modal">
      <small id="close">(esc)</small>
      <h5 id="modal-title"></h5>
      <pre id="modal-content"></pre>
    </div>
    </body>
    </html>
  `;

  var colors = '';
  var sortedColors = insertionSortForColors([...map.keys()], map);

  // Template for each color item with color data
  sortedColors.forEach((val)=>{
    colors += `
      <div
        class="color"
        style="background: ${val};"
        title="${map.get(val).place.join('\n')}">
        <b>${val}</b>
        <i>${map.get(val).index}</i>
      </div>
    `;
  });

  var data = header + colors + footer;

  return fs.writeFileSync('colors.html', data, 'utf-8');
}
```
> *[stylesheets-visualize.js view raw](https://gist.githubusercontent.com/marchi-martius/a08a6f2bb64f11217323baa475304cae/raw/ac94b0f07eb35acacb02a56a2dfc556a41b6b998/stylesheets-visualize.js)*

You may notice two constants representing header and footer — this is for valid HTML5 markup, basic styles and scripts. The body will be generated in a loop. And here is the generated HTML with colors used in app:

![](/assets/2017-06-13-revise-your-stylesheets-part-1-color-scheme-1_eqxlkiDHb2FbMLI3dgcm4A.png)

How do you like these 50 shades of gray, green, red, yellow and so on? I should say that the application was redesigned from scratch with a brand new color scheme and it should have about 24 colors instead of 306! Now I have a total picture of what is actually happening with stylesheets in this app:

![](/assets/2017-06-13-revise-your-stylesheets-part-1-color-scheme-1_0s54mLNzU-unpEHdC1lZZg.gif)

# Add some action

The next step is refactoring and this is where xray may really help. As I mentioned above, for loop generates simple markup for each color using div as a wrapper with the title, style attributes and inner text (color code and index). But most interesting part happens at the footer const.

The footer has markup for modal, enclosing body, HTML tags and plain javascript. It binds event listeners on click which is needed for opening of a modal window containing list of links with paths to files where particular color was found. On each click script checks for a color element, get its attributes and opens a modal with the prepared info. The same click handler checks if target is ‘xray’ link and sends ajax get request to the application’s server with xray URL, for example `[http://lvh.me:3000/_xray/open?path=/Users/username/appname/app/assets/stylesheets/nv/shared/sandbox.scss:456:46`.](http://lvh.me:3000/_xray/open?path=/Users/username/appname/app/assets/stylesheets/nv/shared/sandbox.scss:456:46.)

![](/assets/2017-06-13-revise-your-stylesheets-part-1-color-scheme-1_QskxCKPZMKRbmE4iKQ8Dkg.png)

You click on the color, then click on the path link and xray opens file in your favorite editor. And that’s it! Just two clicks and you’ll get a particular file, move color to separate variable — profit!

# Pros and Cons

### Pros

* The overall picture of colors, sorted by index

* Jump-to-file action

* Easy sharing of generated file with colleagues (just send it via email or drop into slack)

### Cons

* Jump-to-file currently works only for rails app

* You made decision of which colors to merge depending on your vision, literally

* You still have to do a lot of refactoring on each file, like find-replace and name color variables

# Ideas or to be continued…

In cases when you got a good design with predefined color scheme it can be used for automatization of find-replace actions. It would be cool to select few similar colors at the generated `colors.html` and make one-click-merge just selecting one of the colors from predefined color scheme. This means you should prepare separate sass file with named variables for each color in designed scheme. And backend will find-replace all the needed files without your help. Of course, you should use git and commit more to have a changing history. And, yes, I’ll need to run a standalone server on NodeJS and then I’ll write my own xray with blackjack no limits to app/web frameworks.

# Afterword

Yes, you’ll have to make a choice of what shades of colors should be merged. And, yes, it’s not that easy to reduce *>300* colors to *<30*. But at least, you have the overall picture and a handy tool help reducing find-replace operations.

### Share, Star, Blame, Contribute

You’re welcome to submit an issue, pull-request or star [stylesheets-colormap-generator](https://github.com/mir4a/stylesheets-colormap-generator) project on github.


