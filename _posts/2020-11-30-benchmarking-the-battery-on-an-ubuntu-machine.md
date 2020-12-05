---
layout: post
title: "Benchmarking battery life on an Ubuntu machine"
date: 2020-11-29 17:47:52
image: 'BatteryBench/cover.jpg'
share_image: 'BatteryBench/cover.jpg'
description: "Harder than it should be, but easier with a little help"
tags:
- Linux
- Ubuntu
- Battery life
- Optimizations
categories:
- Hacks of life
twitter_text:
---

Since I moved to Linux [some time ago](/moving-out-of-macos-making-linux-habitable/), battery life has been at the top of my list of optimizations. Linux does not have a great history with battery life, and while things have gotten better, the out of the box experience still leaves a lot to be desired. [Here's a post](https://unix.stackexchange.com/questions/119606/why-does-linux-have-poor-battery-life-by-default-compared-to-windows) from 2014 talking about why Linux has a hard time with battery, and things [aren't much better in 2018](https://www.reddit.com/r/Ubuntu/comments/9oyfqd/terrible_battery_life/). With [some manufacturers offering first-party support for drivers and the OS](https://www.forbes.com/sites/jasonevangelho/2019/09/05/dell-has-a-new-dedicated-landing-page-for-ubuntu-and-rhel-certified-linux-desktops-and-laptops/?sh=2a57f27f100f), things are better in 2020 but we're still far perfect.

Now the biggest problem with optimizing battery life is knowing when things are better. The first thing to do is to get yourself a power monitor. Most laptops report their power consumption by measuring the voltage through a resistor, and it's available as a file you can read from inside the OS. Combine this with the current battery level, and you have a simple power monitoring solution. I use [wattmeter](https://extensions.gnome.org/extension/1867/wattmeter/) to display the current usage in watts in the Gnome toolbar, but it's simple to write one yourself to multiply two values together and get an idea.

`cat /sys/class/power_supply/BAT0/power_now` is often recommended to get the power value, but this didn't work on my system. For me, multiplying the current usage (from `cat /sys/class/power_supply/BAT0/hwmon1/device/current_now`) with the voltage (from `cat /sys/class/power_supply/BAT0/hwmon1/device/voltage_now`) yields useful results. YMMV.

This is useful, but it's not very accurate - there's an unpredictable amount of averaging going on (due to variance in how different manufacturers do it, and whether you have good driver support for the specific chipsets involved). Coulomb counting is also not accurate at these timescales, and modern systems ramp power usage up and down many times a second for burst processing, so the exact moment of measurement matters a lot. I've found it's helpful to wait a few seconds, then read the power usage in high/medium/low terms (above 11 W is high, below 5 W is low) to get a general idea.

## Better benchmarking

The best way to benchmark any battery is to run it down under constant, predictable load and see how long it lasts. This averages out all of the variable factors and tells you exactly how much use you can get out of it. Problem is - and this boggles me - Linux does not seem to have good battery benchmarks. I've had no success with Geekbench and the like, and the common advice seems to be to roll your own. This is exactly what I've had to do at the beginning. This single line bash file runs every minute from a cron job, and logs the data to a csv file for later analysis.

```bash
echo "$(uptime), $(acpi)" >> ~/logs/batlog.csv
```

I've had it running for a few months now (and generated a few good megabytes of useful data):

![Battery log]({{site.url}}/assets/img/BatteryBench/batterylog.png)

Problem is, I haven't had the time to build a good analytics and measurement suite around analyzing this data. The eventual goal is to feed it into a simple perceptron to get better battery life estimates, something I did for my old Macbook when Apple's power reporting was a hot mess.

Let's save that for another time. This is about [gnome-battery-bench](https://blog.fishsoup.net/2015/01/15/gnome-battery-bench/), a tool that (embarrassingly for me) has existed since 2015 and does a good job at the few simple things you need for battery benchmarking. It can record and replay events on a time loop to simulate activity, record battery usage, and give you a graph of what that looks like.

Unfortunately, the tool is poorly documented (with conflicting instructions), and I couldn't find an easy way to install it on Debian. After much finagling with dependencies (which are named differently on different distros), I've managed to get it working on my system. Here's how, for anyone searching for the same things as me.

## Installation

Once you've got a method that works, it's actually pretty simple. First, you [download the rpm package from pkgs.org](https://pkgs.org/download/gnome-battery-bench) (I presume you're on x86, although with every other chip manufacturer leapfrogging Intel that may not be true in a few years). Next, you use [alien](https://manpages.ubuntu.com/manpages/bionic/man1/alien.1p.html) to convert the rpm to a deb package, and install that. That's it!

```bash
wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/g/gnome-battery-bench-3.15.4-6.el7.x86_64.rpm
sudo apt install -y alien
sudo alien -k gnome-battery-bench-3.15.4-6.el7.x86_64.rpm
sudo dpkg -i gnome-battery-bench_3.15.4-6.el7_amd64.deb
```

You can now open the GUI (called Gnome Battery Benchmark) or run it from the command line as `gbb`.

## Recording a session

GBB comes with a nifty feature for recording and replaying HID events like mouse and keyboard. This makes it easy to record a common use case for you, and then replay it for 30 minutes or until the battery dies, to see what you would realistically get. This to me is the most useful feature, since I'm usually looking for two things. One, how much battery life can I get with realistic usage, and two, how does swapping out a tool (like Firefox for Chrome) or adding a battery optimization (like limiting processor frequency) affect my battery life.

Recording a session is as simple as `gbb record > <test Name>.loop`. This saves the keypresses and mouse movements to a file that you can use as a test. The contents of the file look a lot like this:

```csv
KeyPress,2488,1732,2675,125 # KEY_LEFTMETA
KeyRelease,2565,1732,2675,125 # KEY_LEFTMETA
KeyPress,3140,1732,2675,33 # KEY_F
KeyRelease,3224,1732,2675,33 # KEY_F
KeyPress,3237,1732,2675,23 # KEY_I
KeyPress,3309,1732,2675,19 # KEY_R
KeyRelease,3331,1732,2675,23 # KEY_I
KeyRelease,3394,1732,2675,19 # KEY_R
KeyPress,3483,1732,2675,18 # KEY_E
KeyPress,3537,1732,2675,33 # KEY_F
KeyRelease,3580,1732,2675,18 # KEY_E
KeyRelease,3639,1732,2675,33 # KEY_F
KeyPress,3652,1732,2675,24 # KEY_O
KeyRelease,3736,1732,2675,24 # KEY_O
KeyPress,3740,1732,2675,45 # KEY_X
KeyRelease,3834,1732,2675,45 # KEY_X
KeyPress,5154,1732,2675,28 # KEY_ENTER
KeyRelease,5245,1732,2675,28 # KEY_ENTER
```

If you'd like to record a session in more detail, you can also create `.prologue`, `.epilogue`, and `.loop` files separately. The prologue is played first, then the loop for the duration of the test, and then the epilogue to wind down and return the system to a pre-test state.

A `.batterytest` file is needed, but it's usually a three line affair with some metadata on what the name and description of the test are. Here's an example:

```
[batterytest]
name=Chrome
description=Just some operations in Chrome
```

Once you have your files, copy them into `/usr/share/gnome-battery-bench/tests/` and you're done. The test should now be in the GUI, or you can invoke it from the command line. If you're fine tuning a test, `gbb play` and `gbb play-local` can also be handy.

## Running a test

Running a test is as simple as selecting a test to run along with conditions under which to run the test. In my experience, the 30 minute test with the backlight at 50% proved to be the right length. Too long, and you're spending 8-11 hours (if you've done a good job) per test. Too short, and you don't give the CPU and hardware enough time to settle into a steady state.

![GBB]({{site.url}}/assets/img/BatteryBench/gbb.png)

## Unresolved issues

There were no showstopping issues for me, but GBB couldn't reliably read power data from my machine. I suspect it has something to do with the weird way AC power and amperage values are reported in my system. Honestly, all I need to answer my questions is the derivative of the charge value over a reasonable enough period of time. This is recorded into a json file (at `~/.local/share/gnome-battery-bench/logs`) for each run of a test, with the number of milliseconds since test start and the current charge:

```json
{
  "test-id" : "firefox2",
  "test-name" : "Firefox browser2",
  "test-description" : "Just the same operations but in Firefox",
  "duration-seconds" : 1800.0,
  "screen-brightness" : 50,
  "start-time" : "2020-12-03 08:47:11",
  "log" : [
    {
      "time-ms" : 0,
      "online" : false,
      "charge" : 2231000,
      "charge-full" : 6284000,
      "charge-full-design" : 6707000
    },
    {
      "time-ms" : 19309,
      "charge" : 2220000
    },
    {
      "time-ms" : 37869,
      "charge" : 2209000
    },
    {
      "time-ms" : 57183,
      "charge" : 2198000
    },
  ]
}
```

### Results

I've so far run three main benchmarks: a simple workload test on Chrome that simulates opening a Youtube video and letting it play in the background while I open google docs and typeracer and play with it for a while before opening more, a firefox workload with the same behavior, and the included `lightduty` load that opens Firefox and Gedit and mucks around for a while.

The lightduty test was allowed to run down the battery until 5%, while the others were run for 30 minutes each. I ran the firefox and chrome benchmarks twice, and the Chrome results were almost exactly on top of each other. Firefox seemed to have varying levels of consumption, but that might be due to Dropbox kicking on in the background. We'll disregard this kink for now, I plan to run more tests.

I whipped up a quick plotly script to plot the charge values from the json, as well as do some basic linear regression to estimate the total battery time. I'm also normalizing the start percentage to 100, so we can put them next to each other. These tests were run with somewhere between 90% to 20% battery remaining. I recommend not running battery tests on either extremes of the charge range - some hardware might have issues with their coulomb counters close to the 100% mark, and additional battery saving measures usually kick in below 15-5%.

Here's what just the data (no fit) looks like:

![Battery Graph without fit]({{site.url}}/assets/img/BatteryBench/batterynofit.png)

Now let's calculate time remaining and extrapolate the last 5 minutes of data to find the end point:

![Battery Graph with fit]({{site.url}}/assets/img/BatteryBench/batterywithfit.png)

Couple of early observations: The chrome data seems remarkably consistent, and the lightduty test with extrapolated data (7 hours 33) is reliably close to the one I ran until 5% (7 hours 21).

But damn if Chrome isn't a battery hog. Doesn't mean I'm switching to Firefox yet, but I'm definitely opening it up on an airplane.

I've <a href="https://gist.github.com/hrishioa/d087ec51c8c5949e6746fe24343ea5d3" target="_blank">uploaded the visualisation script here</a> if anyone's interested. It's a single html file, runs locally, no real styling to speak of. If someone would like to add to it, I'm happy to create a repo!

