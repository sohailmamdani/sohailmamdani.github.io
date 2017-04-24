---
layout: post
title: From Jamf to Chef, Part 1
date: 2017-04-24 00:38 -0700
categories: tech
tags: mac, chef, sysadmin, tech, opensource
---

I've spent the last five years honing my skills as a Jamf admin. I started at eBay, working for one of the best bosses I've ever had (ohai Alex Dale). Since then, I've gotten my CCA and CCE and have had the pleasure of working for a number of companies where I've either implemented or improved on the Jamf framework, imaging workflow, and a whole lot more.

I'm currently working at GoPro as the Senior Client Engineer for the Mac fleet and I'm spearheading our conversion from  Jamf to Chef. It is _easily_ the most challenging thing I've ever done, not least because it is nothing like Jamf, but because you actually have to undergo a mindset change.

Here then, is a somewhat rambling, somewhat contextualized look at my process for moving from the safe, comfortable world of Jamf and into the frontier that is Chef Open Source.
<!-- more -->

### Context
If you've been a Mac admin for long enough, you're likely to remember that your job did not really involved spending time in the weeds of a command-line interface. Being a Mac guy was a very visual thing; you had a tendency to scoff at Windows admins who'd drop into the command prompt for their silliness.

That's the kind of Mac admins that have been in the industry for a long time. The advent of Mac OS X (now macOS) did shift that with its UNIX underpinnings, but Mac admins hung on to their graphical ways for a long time - and still do.

That's the context many of us have operated under. I certainly resisted learning even Bash.

Then, about two months into my time as a Jamf admin at eBay, I decided to learn it so I wouldn't feel stupid everytime I went on Jamfnation to ask a question.

### A Shift in the Ecosystem
Here's something I realized early on: *The visual-centric way of administering a Mac endpoint is a dead-end*. Even though being a Jamf admin still means you spend a fair bit of time  with a graphical UI, I don't know a single person in the Jamf community worth their salt who doesn't have a respectable collection of at least Bash scripts. More often than not, like me, a lot of admins have developed custom workflows in Bash, Applescript, Python, whatever, and are just using Jamf as a sort of API framework (sometimes, we even make RESTful calls to the Jamf API).

So, even though the Jamf system is something of a monolithic black box, by working with the Jamf API and the commands available via the `jamf` binary as a sort of extended API, admins have modularized the Jamf ecosystem.

[Rich Trouton's](http://derflounder.com) awesome [CasperCheck](https://github.com/rtrouton/CasperCheck) script is a classic example of this. The Jamf system doesn't have a self-repair mechanism (ffs guys, why _not_?), so Rich built one. In doing so, he taught me the intrinsic value of using functions in Jamf, then further modularizing code by breaking out all commonly used functions into a separate file that can be stowed on every endpoint and merely referenced with a single `source /path/to/file` line at the beginning of your script.

So Rich made this script. I made a custom version of it that also sends me an email every time a computer can't connect to our JSS with a copy of the `jamf.log` and the user's external and internal IP addresses. I also edited it heavily to log additional data to a custom log file so we can pull that into our ELK infrastructure.

That's one example; I've got more, but you get the idea.

### The Frayed Edges
Ultimately though, Jamf is starting to show some frayed edges. Patch management is one. The unreliability of stored Filevault keys is another. The consistency with which I run into `Device Signature Errors` is a third. And on and on.

Too, there is the price. It's actually reasonably priced, to be fair to Jamf, comparable to other management systems, but it isn't cheap. It's also Mac-only, is a black box (so transparency is kinda hard), and the bugs are, well, bugging the daylights out of me. If I have to see one more already-encrypted machine somehow leak out of the smart group containing all encrypted machines that's set up as an exemption list to our "Encrypt this Mac NOW" policy, I'm gonna need a stiff drink.

Then attempt to add something like DEP to the mix. Right away, you'll notice that when you create Prestage Enrollment profiles, you'll need to find a way to exempt a newly-enrolled Mac from get hit with all the policies you have set up to run at the next check-in (like Encryption). Ditto if you create a custom deployment workflow that can be run on a new-from-the-box Mac. In fact, I had to come up with this little bit of code in my new no-imaging workflow.

```bash
############ Add JSS ID To Exempt Group ###############
echo "50 Adding computer to temporary exempt group" >&3
echo '<computer_group><computers><computer><id>'$jssid'</id></computer></computers></computer_group>' > "/private/tmp/addtogroup.xml"
curl -s -k -v -u "$user:$pass" https://yourjssurl:8443/JSSResource/computergroups/id/XX -X PUT -T "/private/tmp/addtogroup.xml"
sudo /usr/local/bin/jamf recon
LogScript "Added computer to JSS Exempt Group"
############ End Add JSS ID To Exempt Group ###############
```

That computer group had to be added to the exemption list for every single policy and mobileconfig. Yeah. No fun. But it worked.

At the end, I ran this:

```bash
############ Remove computer from Exempt group #############
echo "80 Removing computer from exempt group." >&3
echo '<computer_group><computer_deletions><computer><id>'$jssid'</id></computer></computer_deletions></computer_group>' > "/private/tmp/removefromgroup.xml"
curl -s -k -v -u "$user:$pass" https://yourjssurl:8443/JSSResource/computergroups/id/xx -X PUT -T "/private/tmp/removefromgroup.xml"
echo "90 Cleaning up" >&3
############ Add Remove computer from Exempt group ###############
```

Frayed edges like these drove towards the big change that I was told about during my job interview at GoPro.

Coming next: The Sea Change.
