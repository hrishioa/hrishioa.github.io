---
layout: post
title: "Automating Git Commits"
date: 2016-03-12 20:41:37
image: '/assets/img/'
description:
tags:
categories:
twitter_text:
---

There are any number of tools available for the developer that wishes to check up on his project while he's away. I've made use of a number of these tools on occasion, but recently I found myself wanting something more.

The project I wanted to monitor and keep track of was on a [Beaglebone Black](https://beagleboard.org/black), and from experience I knew that the particular device I was using wasn't the most reliable for data fidelity. I needed a way to store the current state of the project on the cloud. 
It's worth noting that Git wasn't my first thought, as Dropbox or any other cloud hosting tool can easily provide this service. But what if you needed version control as well? Even on a binary blob that's being constantly updated, it was useful for me to have rollback - especially when it's time based rollback. What if I could move back four commits and be forty minutes into the past? Now that's something. (Also I was running out of space on my Dropbox)

