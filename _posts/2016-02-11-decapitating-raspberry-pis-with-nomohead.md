---
layout: post
title: "Decapitating Raspberry Pis with nomohead"
date: 2016-02-11 05:24:08
image: '/assets/img/'
description:
tags:
- Raspberry Pi
- ngrok
- IoT
categories:
twitter_text:
---

The Raspberry Pi is one of my favorite SoCs out there. It's cheap, it's everywhere and it practically runs anything. With the new update, it's faster than ever and possibly everything I could want in most of my projects. 

![raspi-img]({{site.url}}/assets/img/nomohead/piimg.jpeg)

The only problem is set-up. I rarely ever need to run a pi other than in headless mode, and once the IP addresses are set up I mostly work through ssh. Occasionally I use VNC if I need to look inside the pi, but I rarely need to connect the Pi to a monitor and mouse/keyboard. Except for set-up.

I live in a [university](http://www.yale-nus.edu.sg/), where the routers refuse to give you any kind of access that would help you locate your pi, much less talk to it. This means that every time I power up my Pi, I need to find the nearest 'head' so I can get the IP (no static IPs. None.).

There are a number of workarounds for this, including using a DNS service to having the Pi display it on a small set of seven-segments, but I've settled on one for now (the link is below), and it solves most of my problems.

It's a little script that is easy to install, lets the Pi boot up, and then [dweets](http://dweet.io) the ngrok address and ip to a preset id. It's come in handy more than once, and I'm thinking this is going to help you quite a bit, especially if you go to hackathons and have no idea what the wifi situation will be like.

![dweet]({{site.url}}/assets/img/nomohead/dweet.png)

It's saved my ass twice this week already.

Here's the code and the installation script - [nomohead](https://github.com/hrishioa/nomohead)