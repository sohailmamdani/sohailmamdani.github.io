---
layout: post
title: Blogging With Github Pages
date: 2015-12-22 17:24 -0800
categories: tech
tags: writing,blogging,github,markdown
---

Github Pages + Jekyll + Markdown + Typekit + TextExpander = Bliss. 

I'm liking the combo of Jekyll and Github Pages a LOT. Just added Typekit to my site so I can use the excellent [Lato font](http://www.latofonts.com/lato-free-fonts/) by Warsaw-based [type designer Łukasz Dziedzic](https://plus.google.com/106163021290874968147/posts). I may have gone a bit nuts with the Typekit embed code and added it to not just the head.html in my includes directory, but also to all the templates and other pages. Sorry if this causes any issues.

<!-- more -->

Also exploring using redcarpet markdown instead of kramdown so I can use Github-flavored markdown. This is mostly so I can mess around with `inline code`. Wonder if this works with standard syntax highlighting.

Github Markdown test:

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```
 
Standard Jekyll Syntax:

{% highlight json %}
var s = "JavaScript syntax highlighting";
alert(s);
{% endhighlight %}
 
Using the Github Markdown definitely works better - and I can preview the syntax highlighing in the Github web interface too, which is not the case for Jekyll code. 

Next up, Disqus integration. Because I know you're all dying to interact with me via my blog.

Then: truncating posts on the front page.
