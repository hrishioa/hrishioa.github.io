---
layout: post
title: "Moving out of MacOS: Making Linux habitable"
date: 2020-08-09 20:51:06
image: 'MacToLinux/cover.jpg'
share_image: 'MacToLinux/cover.jpg'
description: It's better with a little TLC
tags:
- Linux
- Ubuntu
- MacOS
- XPS13
- Personal
categories:
- Hacks of Life
- Guides
twitter_text:
---

With a lack of hardware updates, buggy keyboards that never get fixed, removed SD card slots and useless touchbars, the Macbook ecosystem has been languishing for a while. Apple seems intent on cutting away the long tail of power-users, be it audio professionals, video editors or developers. With the recent ARM-transition and Big Sur, it was clear to me that Apple and the Macbook were moving in a direction that I couldn't join them in, and I refuse to be dragged to along. An iPad with a keyboard running on an ARM chip might honestly be a great computer for the vast majority of users, but I don't see my requirements fitting into that box - at least for the next five years. I might do a second part on why I left and what the other side offered, along with the choices encountered and an evaluation of the options, but I'll cut it short here by mentioning my final choice: Ubuntu 20.04 running on an XPS 13 9300 Non-Developer Edition.

The Linux ecosystem is a lot better than it was when I left five years ago, but it still has a ways to go. I've had to do quite a bit to get my system to a point where I can reliably and comfortably work on my computer, without spending too much time *working on my computer*. Don't get me wrong, I do quite enjoy recompiling kernels and tweaking my workflow - and Linux gives you the most control there - but I still do want a machine I can feel at home in, and not a permanent work-in-progress. Here's a guide on what I ended up doing to port over the functionality I missed from the Mac and additional improvements made. It was a day or two of work but in the end, I'm happy to say that I haven't felt the urge to open up my Mac since I finished, despite it being my happy home for the last half-decade or more.

We'll go through setting up biometrics, backups, gestures, optimizing battery life, and some good alternatives to tools you'll need.

## First steps

Before you get started with the specifics, doing some things first will speed up the process. The tools to work on the tools if you will, and some creature comforts.

### Gnome Shell Extensions

Ubuntu uses Gnome 3, which (despite the reports I see online) has been a relatively light and power-sipping window manager in my use. It's the Goldilocks spot for me - you'll have better battery life and speed with LFCE, more customization with KDE, but Gnome is yet to give me a reason to leavel .Gnome Extensions make it a lot more customizable and powerful, so the first thing you'll want to do is to install [the Gnome Tweak Tool](https://linuxconfig.org/how-to-install-tweak-tool-on-ubuntu-20-04-lts-focal-fossa-linux). Don't forget to install the [browser extension](https://chrome.google.com/webstore/detail/gnome-shell-integration/gphhapmejobijbbhgpjhcjognlahblep?hl=en) for Chrome (or the browser of your choice), it makes installation and management painless.

Once that's done, you'll want to install a nice theme and set up a few extensions. Here's what I'm running:

![theme]({{site.url}}/assets/img/MacToLinux/theme.png)

1. [Flat Remix Blue Theme](https://www.gnome-look.org/p/1214931/) - simple aesthetic preference but I like the transparency (if it has an impact on battery life I'm yet to see it), and the blue accents.

2. [Workspace Matrix](https://extensions.gnome.org/extension/1485/workspace-matrix/) - I held off on this one for a while, hearing about issues but I've been a happy camper ever since I switched. Set rows to 1 for a more Mac-like experience with workspaces.

3. [Caffeine](https://extensions.gnome.org/extension/517/caffeine/) - very similar to the one I had on MacOS, keeps your device from going to sleep. Turn it on when you've got something you'd like to run uninterrupted.

4. [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/), [NetSpeed](https://extensions.gnome.org/extension/104/netspeed/), [Wattmeter](https://extensions.gnome.org/extension/1867/wattmeter/) and [CPUFreq](https://extensions.gnome.org/extension/1082/cpufreq/) - helpful if you'd to interact with the clipboard, CPU frequency governors and keep track of the internet speed. Netspeed comes in handy when you're trying to find out whether your program or your connection is being laggy. Wattmeter doesn't do anything intelligent - it simply multiplies the amperage used by the voltage to get Watts, but it's been useful in letting me know when Dropbox is eating all of my battery.

5. [Panel OSD](https://extensions.gnome.org/extension/708/panel-osd/) - I hate the default position and background of Ubuntu notifications, and Panel OSD helps move them to a less obtrusive location.

6. [Transparent Top Bar](https://extensions.gnome.org/extension/1708/transparent-top-bar/) - I like transparency. I'm one of those people that actually *liked* Aero effects.

7. [Dash-to-Dock](https://extensions.gnome.org/extension/307/dash-to-dock/) - I wouldn't personally recommend dash-to-dock. It's crashed on me pretty regularly and hiccups on multi-monitor setups. But the rest of the world seems to be having a better experience than me - and it's great when it works - so it's worth a try.

### Install a terminal emulator (and a shell)

Nothing wrong with the one that comes in the box, but if you're expecting to spend any time in the Terminal, I would recommend [Tilix](https://gnunn1.github.io/tilix-web/). Pretty lightweight, all the settings I was looking for, and it supports Quake-style dropdown terminal which is a godsend on a small laptop screen.

I also installed [Zsh and oh-my-zsh](https://dev.to/mskian/install-z-shell-oh-my-zsh-on-ubuntu-1804-lts-4cm4) that I'd gotten quite used to on MacOS. While I initially hated the reason Apple moved away from Bash, Zsh won me over with its innocent features. [Powerlevel10K](https://github.com/romkatv/powerlevel10k) has one of the best set-up processes I've ever seen, and [gitstatus](https://github.com/romkatv/gitstatus) gets rid of lag when opening a complex git directory. I use node on a regular basis, and adding `--no-use` to the nvm loading script in `~/.zshrc` (or `~/.bashrc`) sped things up quite a bit. Tilix also needs to use the Virtual Terminal Emulator which zsh doesn't add by default, but [adding it is quite simple](https://github.com/gnunn1/tilix/wiki/VTE-Configuration-Issue).

### Set up gestures

I'm placing this early so you don't have to go through what I did. I had a lot of trouble setting up reliable gestures to port over my muscle memory from MacOS. I failed, gave up, came back and eventually managed to do it. If you do this from the beginning, you won't need to go through the repeated re-adaptation I needed.

A number of utilities exist for gesture support on Linux and Ubuntu, and I've tried most. [Fusuma](https://github.com/iberianpig/fusuma) was the one that finally worked for me. I'd recommend setting up Workspace Matrix (see above) for a horizontal workspace configuration. Not only is this similar to MacOS, but the up and down gestures are often prevented by applications like Chrome, which seem to want to capture and keep the event for themselves (why this is I have no idea). This means that for three-finger swipe, left and right is far more reliable than up and down.

Best runner up is [comfortable-swipe](https://github.com/Hikari9/comfortable-swipe), which worked well but proved slightly unreliable with gesture recognition. When you're trying to build in muscle memory, I found that 90-95% wasn't good enough - your brain simply goes back to the other way of doing it.

I recommend installing the [sendkey plugin](https://github.com/iberianpig/fusuma-plugin-sendkey) for fusuma, which has lower latency compared to the default [xdotool](https://github.com/jordansissel/xdotool). Pinch and four-finger swipes also work well. Here's my configuration:

```yaml
swipe:
  3:
    left:
      sendkey: "LEFTCTRL+LEFTALT+RIGHT"
      threshold: 0.2
      interval: 1
    right:
      sendkey: "LEFTCTRL+LEFTALT+LEFT"
      threshold: 0.2
      interval: 1
    up:
      sendkey: "LEFTMETA"
      threshold: 0.5
      interval: 1
    down:
      sendkey: "LEFTMETA"
      threshold: 0.5
      interval: 1
  4:
    left:
      command: "xdotool key XF86MonBrightnessUp"
      threshold: 0.2
      interval: 1
    right:
      command: "xdotool key XF86MonBrightnessDown"
      threshold: 0.2
      interval: 1
    up:
      sendkey: "LEFTMETA"
      threshold: 0.5
      interval: 1
    down:
      sendkey: "LEFTMETA"
      threshold: 0.5
      interval: 1
pinch:
  in:
    sendkey: "LEFTCTRL+EQUAL"
    threshold: 1
    interval: 0.5
  out:
    sendkey: "LEFTCTRL+MINUS"
    threshold: 1
    interval: 0.5
```

You'll notice I use the four-finger left and right swipes for brightness controls.

![workspaces]({{site.url}}/assets/img/MacToLinux/workspaces.gif)

### 4. Development Environment

[Micro](https://github.com/zyedidia/micro) is a good upgrade from nano (if you're not already on emacs or vim), with clipboard support, good syntax highlighting and plugins. Makes editing the odd commit message or crontab easier.

I'll also mention VS Code, as porting over settings and extensions took a few extra steps. Proper [Settings Sync](https://code.visualstudio.com/docs/editor/settings-sync) isn't in the stable build yet, so if it still isn't when you're reading this, the extension to [sync settings through a gist](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync) is a workable option.

### File Explorer

This is entirely personal preference, but I'd recommend [Nemo](https://itsfoss.com/install-nemo-file-manager-ubuntu/) over the default Nautilus on Ubuntu. It's got more settings, looks better (to me), and loads faster on my system. If you're a Dropbox user and you go this route, you'll also need [nemo-dropbox](https://github.com/linuxmint/nemo-extensions/tree/master/nemo-dropbox) for better integration - although Ubuntu 20.04 doesn't seem to have proper support just yet. Not an issue - you can install it from the [.deb release](https://pkgs.org/download/nemo-dropbox).

## Backups

The next big thing for me was configuring backups. From past experience, setting up backups is a lot easier on a clean system and a lot easier to forget once you're up and running. There are a number of good options out there, but I'll rip the bandaid off: I couldn't find one that was as good as Time Machine. If you're looking for something that can backup your entire system into a bootable copy while it's running, you'll have to cobble the functionality together in pieces. If anyone knows anything better than what I'm suggesting here, please do reach out - you'll make my life easier.

For timed backups and archive management, you need something that can keep incremental (or differential) backups and prune old ones. It's entirely possible to set something up with [rsync](https://linux.die.net/man/1/rsync), but that comes with its own problems. If you roll your own, you either have to be very diligent in setting it up or make sure you have enough reporting in place. You don't want to discover that your backups have been silently failing for the last four months, and you certainly don't want to discover it when staring at a dead machine.

[Borg](https://borgbackup.readthedocs.io/en/stable/) has been the best solution for my case, with [Vorta](https://github.com/borgbase/vorta) as the GUI. It's got something of a learning curve, but once it works, it works well. I'm using a Synology NAS at home running RAID as the main onsite backup, and using an SSH Repository worked best for me. Simply mounting an NFS or Samba share will also work, but you then need to take care of making sure the volumes are mounted on boot (which isn't always possible depending on where you are) or the backups can fail.

[Timeshift](https://github.com/teejee2008/timeshift) is rsync based and a lot easier to use, but I found it failing silently now and then with no real reason.

On top of this, I kept the original Windows installation around (despite my optimizations, Windows is still better at battery life - averaging about 14-16 hours to my optimized 9 on Ubuntu) for media or long flights, and using [Macrium Reflect](https://www.macrium.com/reflectfree) to take a full image of the entire drive once every two months or so is a good idea if you choose to do this.

(One additional point worth mentioning if you're on the XPS13 is that it has a MicroSD card slot. If you leave a big one (256GB-1TB) plugged in, it makes a good quick backup drive to go back in time or detach when you leave home!)

## Biometrics

Biometrics were a little bit of work to set up - the story may be different on a proper Developer Edition XPS13 - but they're great once you have them set up. Sudoing in with your face or your finger still hasn't stopped being cool to me.

For fingerprint authentication, [fprintd](https://launchpad.net/~fingerprint/+archive/ubuntu/fprint) is what finally worked for me without much trouble - but it's worth reading through the linked page for exceptions - but YMMV. You may need to check the supported devices list or check `lsusb` to see if fprint can see your fingerprint reader.

[Howdy](https://github.com/boltgolt/howdy) works great for facial recognition. I had to change `device_path=/dev/video2` to get it to read the right camera. If you have problems, install [V4L2](http://manpages.ubuntu.com/manpages/xenial/man1/v4l2-ctl.1.html) with `sudo apt install v4l2-utils` and run `v4l2-ctl --list-devices` to find your camera driver. Add it to `sudo howdy config` and you should be good to go. I made some additional tweaks to the facial recog configuration to make things run smoother and I'll list those values below. Keep in mind that some of these trade speed for security, so use at your own risk.

```bash
detection_notice = true # For debugging - but also for coolness
use_cnn = false # Slower without much increase in accuracy
certainty = 3 # Good middle ground for security and speed
timeout = 3 # Most recgnition will fail within 2 seconds, so this is good
force_mjpeg = true # Use this for debugging, or `howdy test` will often break
```

## Battery Life

Also known as the bane of Linux - something I knew I'd be taking on when I switched over. When I said that Linux had come a long way and still had a lot to go, I was talking about battery life. Everything else - mostly everything else - is now as good or better than competitors, provided you're willing to put some time in. Even if you don't, most things work well enough out of the box. Except for battery life. Don't stop reading yet, it's not entirely horrible.

As you read through, remember that the tested battery life for the XPS 13 I'm using on Windows is 12 hours - and I've managed to get upto 14 or even (on one rare occasion) 16 hours out of it. Ubuntu out of the box gave me 6. So begins my journey.

The oft-recommended install to fix this is [powertop](http://manpages.ubuntu.com/manpages/xenial/man8/powertop.8.html), and it does help somewhat. But it's mostly a monitoring tool, and most of its recommendations were already followed by my distro - Ubuntu's gotten smarter. The first thing I had to do was write a quick script to log battery level and current uptime to a csv, so I know it's not a feeling that nothing's changed. Here's the script:

```bash
# simple-battery-logger.sh
echo "$(uptime), $(acpi)" >> /home/hrishioa/Downloads/batlog.csv
```

Adding this to the crontab logs the battery every minute, and doesn't consume much by itself, like so:
```bash
* * * * * ./simpleLog.sh
```

I'll admit that it could get better, but it gets the job done and didn't slow me down. When I started testing, most of the battery life optimization tools for Ubuntu did nothing that stood above noise in my measurements, and I won't mention them. The first (and the last) to make a real difference is [TLP](https://linrunner.de/tlp/). TLP doesn't do much out of the box, and you'll need to dig into the configuration to make any actual changes. That's where [TLPUI](https://github.com/d4nj1/TLPUI) comes in handy.

![tlpui]({{site.url}}/assets/img/MacToLinux/tlpui.png)

With a handy explanation of each config right next to it, I'd recommend setting aside 15 minutes to go through all the options and optimize based on your preference. Getting rid of bluetooth for example will make a good difference. From my research, I'll mention that the powersave CPU governor doesn't [do much](https://unix.stackexchange.com/questions/439340/what-are-the-implications-of-setting-the-cpu-governor-to-performance) these days, and it was true in my testing. Ondemand is your best option. Turning off unused cores will help, but it depends on the workload. I find that using CPUFreq (listed above) to adjust cores and frequency based on my workload helps the most. I know what I'm planning to do for the next few hours, be it development or watching movies or reading, and it's something automated systems are yet to be good at predicting.

However, with good configs that didn't really affect system performance, my battery life went from 6 to a good 9 (max 10) hours. Good enough for me - especially right now with Covid-19. Big brain time - you don't need battery if you don't leave the house.

(TLP and CPUFreq might have problems changing the CPU frequency and governor, and if this happens to you, [this might be a useful place to start](https://askubuntu.com/questions/1187564/cant-control-cpu-frequencies-in-19-10-on-7390-xps-13-2-in-1).)

## Utilities

### Screen Recording

[SimpleScreenRecorder](https://www.maartenbaert.be/simplescreenrecorder/) has worked best in my experience. Takes a little setup, but it's the only one I've used that reliably captures the speaker and microphone audio to the recording. Resizing, codec optimizations and GPU optimizations are a bonus.

### Screen Capture

[Flameshot](https://flameshot.js.org/#/getting-start) works best out of the alternatives I've tried. Keyboard shortcuts to capture part of the screen or the full screen, with a fully-featured editor and save to clipboard are all present - and I don't find myself missing Cmd+Shift+5 at all.

For reference, my configs set PrtScr to `flameshot full -c -p <Screenshot_Directory>` and Ctrl+PrtScr to `flameshot gui -p <Screenshot_Directory>`.

### Webcam Capture

I only really use it to make sure I don't look homeless before a call, but [Webcamoid](http://webcamoid.github.io/) is great.

### Zoom

I'll mention Zoom here - not because it's the best video calling software, but because it's common and a little buggy on Ubuntu's QT4. The scaling is off out of the box, so changing the pixel ratio in the Zoom config or the startup command [as detailed here](https://www.reddit.com/r/Zoom/comments/hat5af/linux_client_ui_elements_too_large_after_update/) helps.

I'd also recommend not installing it from the snap store (which is a dumpster fire that is still throwing system errors while fighting with AppArmor - [read here](https://forum.snapcraft.io/t/solved-permission-denied-in-general-ubuntu-19-10-snap-2-42-5/15161) - stay away from snaps if you can). Get the deb [directly from Zoom](https://zoom.us/download?os=linux).

### Database Explorer

Most of everyone reading this will not need one, or have a choice in mind. I'm just adding this to say I miss you, [Postico](https://eggerapps.at/postico/). You will always have a special place in my heart. If you're looking, [pgAdmin4](https://www.pgadmin.org/download/) or [dBeaver](https://dbeaver.io/) work well, just not as well as Postico did \*sob\*.

## Problems I'm yet to solve

There are still a few things that I haven't found a solution for. If you're reading this and you know something, say something.

1. HEVC encoding for screen recording - `ffmpeg` does well above 1 fps on my system for encoding [HEVC](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding) or H.265, so it's possible. I still haven't been able to find software that does this without a lot of work from my end.

2. Palm Rejection - The `libinput` and `synclient` palm rejection settings work well enough for me, and it's been happening less as I adjust to the new hardware. That said I'd love to hear about any solutions out there that can help me customize this further.

3. Notes in the menu/top bar - This is something I sorely miss from my last machine. The ability to quickly add notes through a popup in the menu bar is the only reason I kept Evernote around. All the solutions I've been able to find for Linux have been modeled after Windows Vista's Sticky Notes - not my thing. If there's any tool that will give me an icon in the top bar, that I can click to write things down and persist to disk, I'd love to hear about it before I give up and write my own.

4. Brightness adjustment on XPS13 - this might be very specific to my situation, but controlling brightness from shell has proven problematic. [xbacklight](https://github.com/tcatm/xbacklight) and [light](https://github.com/haikarainen/light) both fail - xbacklight has trouble working with my backlight drivers, and light sets the brightness correctly for a brief second before the system takes over and resets. I have a workaround for now with simulating a brightness key press, but something better should surely exist.

## Conclusions

I'll admit that all of this took some time, but in the end I'm quite happy with it. It's settled into an equilibrium where I'm simply using my computer as a tool and not reaching in all the time to chisel it more. Once you get to this point, Linux really shines. You're not locked to hardware anymore, and for developers - especially backend - actually working on the same kernel as the one you deploy to is a major bonus. The system is as snappy as MacOS is, and on the XPS 13 the keyboard, display and industrial design are miles ahead. I'd give a slight edge to the Macbook trackpad, and a strong edge to the Macbook speakers, but they're still pretty great. No more mucking around with brew and macports and competing versions of ruby and python, no more touchbar. Life is good.

This is not to say that I'm leaving the Apple ecosystem for good. If - and when - Apple can prove that it still cares about the Macbook (beyond turning it into a heavier iPad) and the developers who use it for non-Apple development, and they regain the giant lead they once had in hardware, I'll happily switch.