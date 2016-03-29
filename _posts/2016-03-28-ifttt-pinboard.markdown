---
layout: post
title: IFTTT and Pinboard (and why I maybe need to learn to code for reals now)
date: 2016-03-28 13:24 -0800
categories: tech
tags: tech,web,business
---

I'm browsing through emails this morning, and this little gem pops up from [IFTT](http://ifttt.com).

![](img/2016-03-28-ifttt-email.png)

Well that sucks. I pipe all my "Saved for later" items from [Feedly](http://feedly.com) into Pinboard, and I have Pocket send links to all saved articles to Pinboard via IFTTT too. 

WTF. The email from IFTTT made it clear that Pinboard was to blame for this. 

So I sent off an angry tweet to Pinboard. 

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">.<a href="https://twitter.com/Pinboard">@Pinboard</a> How come you guys aren’t moving to <a href="https://twitter.com/IFTTT">@IFTTT</a>’s new platform? I use the Pinboard channel regularly and depend on it.</p>&mdash; Sohail Mamdani (@sohail) <a href="https://twitter.com/sohail/status/714540088630202368">March 28, 2016</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Then I went looking for a solution and came across a few tweets from Pinboard that shone some light on the issue.

<!-- more -->

Here are the ones of interest, oldest to newest. I haven't quote all of his tweets in reference to this story, just the ones I felt were most germane to the topic.

<div class="storify"><iframe src="//storify.com/SohailMamdani/the-ifttt-pinboard-story/embed?header=false&border=false" width="100%" height="750" frameborder="no" allowtransparency="true"></iframe><script src="//storify.com/SohailMamdani/the-ifttt-pinboard-story.js?header=false&border=false"></script><noscript>[<a href="//storify.com/SohailMamdani/the-ifttt-pinboard-story" target="_blank">View the story "The IFTTT/Pinboard Story" on Storify</a>]</noscript></div>

Maciej Ceglowski (the creator of Pinboard) also [wrote a blog post](https://blog.pinboard.in/2016/03/my_heroic_and_lazy_stand_against_ifttt/) that goes into some detail about his decision. I dig that he calls it his "heroic and lazy stand" against IFTTT.

Justified stands or not, that kind of leaves me looking for an option. [Zapier](http://zapier.com) is a good one and does what I want, but their free tier has 100 actions/mo and runs every 15 minutes, so my integration would run out of available "zaps" in... let's see, carry the 2... one day. I'd be done in one day. I'd be happy to pay a few bucks for the service but but by the looks of it, the cheapest I could get away with is the [$50/mo service](https://zapier.com/app/pricing).

I did ask for clarification in case I'm wrong, so let's see what they come back with.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/zapier">@zapier</a> could you be more specific about what constitutes a “task” vs a “zap?” Free tier looks like it runs out of tasks in 24hrs.</p>&mdash; Sohail Mamdani (@sohail) <a href="https://twitter.com/sohail/status/714614873267658752">March 29, 2016</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

[Botize](http://botize.io) is the other service recommended by Pinboard, but they don't support [Feedly](http://feedly.com), so that doesn't help me either. 

I thought briefly about leaving Pinboard for another service; maybe [Instapaper](http://instapaper.com) could be my dumping ground for links, or maybe I could go back to [Delicious](http://delicious.com). I even set up a new account with Delicious, but when I went to [http://delicious.com/settings](http://delicious.com/settings) from two different browsers, I got a 404. So, yeah, that wasn't happening.
 
But then I came back to Maciej's tweets and blog post above. I've paid for Pinboard, and I like the tool. It's fast, has a decent API, and it works. 

That got me thinking. 

It has an API. So does Feedly. I've written some stuff in BASH to interact with the [JSS API](http://jamfsoftware.com/developer-resources) as part of my job so I have a basic understanding of how REST APIs work; how hard could it be to build a small script that runs on a cron job or something that interacts with both systems? 

Learning to program is something I struggle with. When I have a job to do, like figuring out how to check for and remove users from the sudo group on a computer if they aren't authorized, I can buckle down, figure that out. I write BASH code, then rewrite it to make it better, and I find that I learn pretty well that way. 

Having a task, with necessity as the driving force behind it, usually works for me. 

So why couldn't it work now? 

I guess it should. Which means I guess I'm about to find out.