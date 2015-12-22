---
layout: post
title: When a Single Line Screws With Your Entire Config
date: 2015-12-22
categories: tech
---

{% highlight json %}
 ssl => true
 {% endhighlight %}
 
 That's it. That one dang line had me running around in circles when trying to implement Filebeat. Because SSL is disabled by default in Filebeat. 
 
 Facepalm.
