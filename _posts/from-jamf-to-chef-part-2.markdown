---
layout: post
title: From Jamf to Chef, Part 2 - The Sea Change (and figuring out a few basics)
date: 2017-05-04 00:38 -0700
categories: tech
tags: mac, chef, sysadmin, tech, opensource
---

"What I want," Darren said, "is for everything, all configuration data, to be text files."

To get the full effect of that sentence, you have to imagine it being said with a British accent, in a voice so low it often feels like he's letting you in on a secret, and with pauses at least three seconds long in place of each of the commas.

This was day two of my employment at GoPro, and Darren, my boss, was explaining his vision to me. Since that time, I've come to realize a couple of things.

1. Darren likes to make short, declarative statements and then wait for your reaction.
2. My time as a Jamf admin is nearing its end.

Oh boy. Here we go.

#### But first: Setting the stage and the timeframe for all this

Here's the deal with these blog posts: they're sort of happening in near-real-time. There's some history from the last six months or so, but most of this is happening *as I write this*. I'm learning Chef as I write these posts. I'm learning Ruby as I type these words out. I'm realizing just how much I don't effing know as I commit these posts to my Github repo.

We have until the end of this year to implement Chef/Munki/Chocolaty/Imagr/MicroMDM/&lt;insert your favorite open source management system here&gt;. That's when our Jamf license expires. We would've done it sooner, but I was derailed by the rollout of our logging infrastructure, which is now complete for all Macs (and which will be the subject of another series of blog posts, I guess).

So, we're now back to Chef. And my panicked realization that I have a LOT of learning to do.

Which is really one of the reasons I'm writing these posts; I've found that actually narrating my experiences as I go crystalizes them. So, in that regard, they're kind of like a journal. Therefore, there's a good deal of exposition interspersed with technical details in these posts.

Just FYI.

#### And now, on to the Sea Change

In [Part 1](http://lowlyadmin.com/tech/2017/04/24/from-jamf-to-chef-part-1/) of this 343-part series, I made a statement of my own. I said:

> The visual-centric way of administering a Mac endpoint is a dead-end.

Now here I was at GoPro and that I was going to have to make good on that idea in the most complete way imaginable.

I needed a starting point. So I reached out to some really awesome guys at the Client Platform Engineering team at Facebook. They invited me and my boss over for lunch and we had a good, long chat about our plans.

In that conversation, one of the most important and relevant questions I asked was, "I'm a Jamf admin. Tell me what I need to change about my mindset as we move forward with this project."

That set Nate and Mike back a bit. They took a few seconds to answer. I don't recall the exact words, but here's the gist of what I remember: "Forget what you know as a Jamf admin. There are a lot of bad habits to unlearn. Come at this from the perspective of learning something completely new."

There was more, but I don't recall all of it. I may have been too busy visualizing all of the value of my past experience slowly circling the drain.

Oh - Mike said something about idempotence. I remember that too.

That's a freaking sea change if ever there is one.

Welp, there was nothing to it. I just got to it.

#### Finding my sea legs

![][seachange]

(I'm awesome at bad analogies but I think this series of nautical-themed ones are working so far. Inevitably, I'll take it too far.)

The first thing on my agenda after coming back from eating tacos and ice cream at Facebook is to sit and down parse exactly what Chef would - and would not - do. *This is the part that I think current Jamf admins will find most relevant*.

Things I learned:

- Chef is *NOT* a replacement for Jamf.
- Chef *IS* a replacement for some of the components of Jamf.

Let me expound on that a bit. If you're expecting Chef to do EVERYTHING that Jamf does - MDM, software installation and patching, provide a Self Service-like app store, all of that, then you'll be disappointed. That's not the best way to leverage Chef's strengths.

##### So wait - what does Chef do, exactly?

Chef describes itself as an "infrastructure automation" system. Here's how they describe it:

> Whether you have five or five thousand servers, Chef lets you manage them all by turning infrastructure into code. Infrastructure described as code is flexible, versionable, human-readable, and testable. Whether your infrastructure is in the cloud, on-premises or in a hybrid environment, you can easily and quickly adapt to your business’s changing needs with Chef.

Translation: Chef automates process of setting up and configuring your servers (or endpoints, in our case).

What does that mean for Macs?

Well, here's how I see it - and y'all should feel free to tell me how wrong I am. "Infrastructure Automation" in Chefspeak translates to "Configuration Management" when you're talking about endpoints. Which translates into those wonderful things called Configuration Profiles on the Mac.

The beauty of config profiles is that in reality, they're just text files, like this one that sets some Chrome preferences, formwatted as XML:

````XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
   <dict>
      <key>PayloadIdentifier</key>
      <string>com.company.chrome.setting</string>
      <key>PayloadRemovalDisallowed</key>
      <false />
      <key>PayloadScope</key>
      <string>System</string>
      <key>PayloadType</key>
      <string>Configuration</string>
      <key>PayloadUUID</key>
      <string>UUIDHERE</string>
      <key>PayloadOrganization</key>
      <string>Company IT CES</string>
      <key>PayloadVersion</key>
      <integer>1</integer>
      <key>PayloadDisplayName</key>
      <string>Set Home Page</string>
      <key>PayloadContent</key>
      <array>
         <dict>
            <key>PayloadType</key>
            <string>com.apple.ManagedClient.preferences</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
            <key>PayloadIdentifier</key>
            <string>com.company.something.UUIDHERE</string>
            <key>PayloadUUID</key>
            <string>UUIDHERE</string>
            <key>PayloadEnabled</key>
            <true />
            <key>PayloadDisplayName</key>
            <string>Google Chrome</string>
            <key>PayloadContent</key>
            <dict>
               <key>com.google.Chrome</key>
               <dict>
                  <key>Forced</key>
                  <array>
                     <dict>
                        <key>mcx_preference_settings</key>
                        <dict>
                           <key>HomepageLocation</key>
                              <string>http://company.okta.com</string>
                        </dict>
                     </dict>
                  </array>
               </dict>
            </dict>
         </dict>
      </array>
   </dict>
</plist>
````
Since it's a text file, guess what? It can be controlled by Chef.

What else? A while back I wrote about deploying [Filebeat on a Mac](http://lowlyadmin.com/mac/2016/06/02/deploying-filebeat-on-macos/) to collect logs and ship them to a central logging server. Well, the configuration for that - and even the binary, really - can be deployed via Chef.

You can also use Chef to set any static preference plist files (such as a Dock plist in the User Templates folder), set up launchdaemons, and even bootstrap Munki, which will provide the software installation and patching features you would need in place of Jamf.

~~In fact, if you wanted to get really fancy, you could leverage Ohai and write custom data as tags to the `filebeat.yml` config file and workaround a pesky bug in Filebeat that sometimes fails to accurately report the computer's hostname~~ Let's not get ahead of ourselves, mmkay?

##### So wait - Chef doesn't install applications and stuff?

Well, it *can*, but that sort of thing is better left to a tool like Munki. This is what I've come to understand and what brings me back to my one of the two points I made above: Chef is *NOT* a total replacement for Jamf, and Chef *IS* a replacement for some of the components of Jamf.

Okay, we've figured out what Chef can and ~~cannot~~ should not do. Now, how do we go about doing it?

I did say this would be a 492-part series, right?


[seachange]: http://lowlyadmin.com/img/sealegs.gif
