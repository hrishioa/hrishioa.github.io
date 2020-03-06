---
layout: post
title: "Home cooked apps"
date: 2020-02-16 21:12:49
image: 'HomeCookedApps/homecookedcover.jpg'
share_image: 'HomeCookedApps/homecookedcover.jpg'
description: Who do we build for?
tags:
- Electronics
- Python
- Javascript
- Ionic
- Continuous Glucose Monitoring
- Hardware
categories:
- Software Builds
- Hardware Builds
- Hacks of Life
- Building Stuff
twitter_text:
---

Robin Sloan's post about [how an app can be a home-cooked meal](https://www.robinsloan.com/notes/home-cooked-app/) really resonated with me, and it's something I've been doing for a long time. Like a whittler gifts figurines, a blacksmith special swords aimed at that special someone, or someone with a 3d printer who decided not to give any other gift until it breaks down, I've been using the things I've learned over the years to try and make the lives of the people around me better. They're always well received - if you do a good job, you've made someone's life better in a way that they couldn't buy or do themselves.

These (I'll call them home-cooked apps) are essentially apps with a tiny user base, mostly unpublished to the rest of the world, and truly built and maintained with love. If I could put the same amount of thought into everything I build, I think I'll consider myself pretty happy.

Here are some of the ones I made over the time I've spent with my partner. Some are useful, some are fun, and most were also because I wanted to learn something new!

## A literal life-saver

I've [covered this one in some detail](https://hrishioa.github.io/glucose-monitoring-a-story-of-papers,-reverse-engineering-and-the-last-10/) before (the second post is still in the works). She's a diabetic attached to a continuous glucose monitor, and at the time her CGM did not have an app or a way to interface with the sensor outside of the lone and expensive reader, which led to a lot of serious situations that could have been avoided. In what became a joint project, we reverse-engineered the sensor, and I built an Android app in Ionic for her that had better graphing, UI and let you analyze trends over long periods of time accurately. She still uses it from time to time. Most of the time it's nice to have, but it quickly becomes a must-have if you ever find yourself needing it.

Why Ionic? I wanted to learn it, and the fine-grained NfcV control you needed to talk to the sensor wasn't present in React-Native just yet. I figured I'd learn Ionic and wait out this React thing. Any time now.

![the app]({{site.url}}/assets/img/HomeCookedApps/Juventas.png)

You can find the app [here](https://play.google.com/store/apps/details?id=com.juventus.app&hl=en), and the code for it [here](https://github.com/hrishioa/Juventas-Ionic). Our paper explaining the reverse engineering and the biomechanics of measuring glucose from interstitial fluid is [here]({{site.url}}/assets/docs/Glucose/CGMStudy.pdf).

## A literal brownie points app

This one's called Wifepoints. It was born out of a long-running inside joke, and the name was in no way intended to be sexist (which I only found out it could be once her parents saw it). We kept joking that we wanted to give each other points for things around the house, and I thought I'd give it a shot.

It was built as a messenger bot because I wanted to learn the tech at the time. The original concept was simple - you could pair with another person, then text the bot to send them points with a reason. The bot does a little NLP on the backend to make the commands feel like a conversation. The other person would then be alerted. For obvious reasons, you could only give points, not take any away. The intended result was to come across something nice the other person intended for you but didn't make any fuss about and give them some points. The chat would serve as a place you could go back to and see all the nice things someone did for you, which I thought would come in handy in those worst moments of a relationship.

The concept worked amazingly well, and we were quickly using it. However, the feature list kept growing and growing. Unlike building things in my professional life, I enjoyed that I could feature-bloat this thing. Who cares? You could add all the little tricks and whistles that you thought were interesting.

An immediate addition was gamification. Your number of points entitled you to a level, and leveling up came with trivial benefits like being able to find out your number of points with better accuracy, actually finding out the reason someone gave you points with some accuracy.

![the app]({{site.url}}/assets/img/HomeCookedApps/wp1.png)

The app also sent you a GIF that was generated from the reason the other person sent, but you couldn't see the actual reason. It quickly became a game to try and figure out how to decipher the GIFs, and how to word your reasons so the GIFs got more inscrutable and interesting.

![the app]({{site.url}}/assets/img/HomeCookedApps/wp2.png)

There was also cat speak, and many more embarrassing features. Let me know if you'd like to try the app, I'm careful about releasing it even though it can support any number of pairs so I'll do so if there's demand.

It wasn't that complicated in the end - just a simple state machine - so here's the full code in a [gist](https://gist.github.com/hrishioa/64c8644a09c3af80030f1e1a690bf520).

## A Mix Tape

Of course I made her a mixtape. This was the simplest of all the projects, but out of the many years we've been together, this is still in the top 3 most frequently used (of course I collect analytics).

This was back in college, and she wanted to find out more about the different music I listened to. I spent an embarrassing amount of time making a carefully put together playlist that spanned four languages and twice that many styles of music, in true High fidelity style.

![high fidelity](https://media.giphy.com/media/J0u1qXVZS4Xny/giphy.gif)

This wasn't the early 2000s, so while a physical disk or CD-ROM would have been sweet, I knew she couldn't get much play out of these things. I was also venturing into frontend and wanted to build something a bit cooler with CSS stuff. I learned some things and borrowed heavily from [Codrops](https://tympanus.net/codrops/2012/07/12/old-school-cassette-player-with-html5-audio/) to build her one. Putting it on the web meant she could cache it on her browser, and while I sent her the files I knew she'd lose them in a week. This one she could bookmark.

![mixtape]({{site.url}}/assets/img/HomeCookedApps/mixtape.png)

I don't think she'd want me to share the full app, but the code (with the playlist) is [here](https://github.com/hrishioa/HerebeMusic/).

## A heart

The final one was, I think, the most special to her. This was a small 3d-printed heart, inside of which was an MSP430 that ran two LEDs. The LEDs ran a brightness curve that simulated breathing (inspired by the old Macbook LEDs), which I'd played with in high school and was covered [here](http://osx-launchpad.blogspot.com/2010/11/breathing-led-effect-with-launchpad.html). It's great when you find an old website that hasn't been lost to code rot. What was new here was that the LEDs had their frequency and timing vary on a continuously changing cycle that wouldn't repeat for 100 years. The idea was to build something that you could turn on and know that every look will show you a configuration you probably won't see again. The contraption is currently her night light, and it's definitely one that makes her smile.

![the thing]({{site.url}}/assets/img/HomeCookedApps/output2.gif)

(On a side note, [ffmpeg](https://www.ffmpeg.org/) is amazing. Since I found it, I've yet to find something it cannot do that I need. Also, the code is [here](https://github.com/hrishioa/Breathy) if you're interested.)

It's super cheesy, I know. But hey, home-cooked apps are allowed to be as cheesy as they want. I've built a lot of them over the years for family and friends, and it's taught me a lot about building apps. Personal projects tend to be messy and unfinished, but when your creations go out to a small group of people you really care about, even the small bugs are important. You don't have the benefit of scale, you can't forget about that 0.001% of your users. You really learn to deploy and take care of things.

It's definitely something I'd recommend to everyone regardless of skill level - I've learned so much building these, and both creating them and the impact of the final results have brought a lot of happiness into the world, a lot of it mine.
