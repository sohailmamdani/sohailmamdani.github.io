---
layout: post
title: Github Pages Changing
date: 2016-03-29 12:13 -0700
categories: tech
tags: github,blogging,markdown
---

A while back, I [talked about changing my blog over to Github Pages](http://lowlyadmin.com/tech/2015/12/23/blogging-with-github-pages/). At the time, I started using the `redcarpet` Markdown engine so I could get prettier GFM markup instead of Jekyll's standard syntax. 

Then this came in the mail. 

> Starting May 1st, 2016, GitHub Pages will only support kramdown, Jekyll's default Markdown engine.

Blargh! No!

But good news followed in the next line.

> If you are currently using Rdiscount or Redcarpet we've enabled kramdown's GitHub-flavored Markdown support by default, meaning kramdown should have all the features of the two deprecated Markdown engines, so the transition should be as simple as updating the Markdown setting to kramdown in your site's configuration (or removing it entirely) over the course of the next three months.

I made the changes to config.yml and sure enough, GFM-style code snippets work.


    ```javascript
    var s = "JavaScript syntax highlighting";
    alert(s);
    ```


This renders as 

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```

So, you know, yay!
