---
layout: post
title: "Intentions and Program Mind-Meld Frustrations"
date: 2019-09-08 00:45:31
image: 'MindMeld/nano.png'
share_image: 'MindMeld/nano.png'
description: "Why some ecosystems stick and others don't"
tags:
- Python
- NodeJS
- Nano
- EMACS
- VIM
categories:
- Rants
twitter_text:
---

I learned something today that my brain already knew. Triple-clicking on a text view - on an editor, terminal emulator or random display - selects the line. A double click does whatever, but a triple click always selects a line. I found that out courtesy of Rachel at [Rachel By The Bay](http://rachelbythebay.com/w/), but my first response was - triple clicks! Isn't that a relic of the past from when Windows XP had that weird setting? Or some holdover from Xerox Parc? Do we triple click anymore? Must be hard!

Turns out we do - or at least I do. I've been triple clicking editors for most of my life expecting the same behavior - across OSes, terminals, entire genres of programs from IDEs to browsers - and not known it. In fact I realise I actually get annoyed when it doesn't work. It just hadn't been filed away as triple-click in my brain. Didn't really have any name, it was just that thing.

Kind of like moving your fingers. I remember trying to learn which finger I was moving based on sensations and signal feeling when I was a kid. I'd move my fingers then try to remember the finger and move a specific one. Turned out to be a bad idea, and in an hour I had no idea how to move any of my fingers - except for when I wanted to do something. As soon as I had an intent, my brain put together all the things I didn't have separate concepts for and my fingers just worked. "Let's try to move the middle finger now" didn't work as well as "birdie", a lot like a triple-click.

It's these features, these things that we've come to expect of our computing worlds that lend legitimacy and make us feel at home with new products. It's the gravity and ever-present physical environment feedback we get from the real world, so much so that when they disappear we feel uncomfortable and irked for no reason we can identify. Ever see those cats that won't stop spinning and pawing in zero-g? It feels something like that.

It's also the reason the bold, brave new UI refreshes and redesigns get users [angry as hell](https://www.reddit.com/r/redesign/comments/9m0d57/reasons_why_the_reddit_redesign_sucks/), and no amount of devs or marketing pointing out that you can still do all the things you love will calm them down. It's the old feeling of someone having been in your office space, and didn't do things right.

![milton]({{site.url}}/assets/img/MindMeld/milton.jpeg)

These things also save us time by connecting us to our machines and allowing things to flow better. Looking back one of the reasons I stuck with NodeJS when it wasn't a great language (at the time at least) was - and you might think this is the smallest thing ever but I disagree - the persistent command memory in the REPL. I use the REPL extensively. I'm still weird with my code writing process (or so I've been told). I pepper my code with print statements instead of using debuggers. I can use a debugger and look through a dump, this is just faster and helps solidify the code execution flow in my mind. I use the REPL to "unit test" little modules and pieces of code before it ever goes into a file. Adding this new request from an external party? Let me run it through in the REPL and see if I can see the actual output before I even think about opening the file it goes in. 

![milton]({{site.url}}/assets/img/MindMeld/dontdothis.png)

Needless to say I live in REPLs. So when I realised that you could press up and get the last command run in the Node console (did that on pure instinct much like a triple click), and that it didn't erase this history when the process ended, AND that it had a long memory, it felt like I was back at 1G again with earth-like air. Not to mention that this simple feature has saved me countless hours in total time spent building things. It's when programs like python don't have that feature - not natively, not in all distributions and if you don't have it as a baseline then you can't *expect* it - that things like ipython become a *necessity* and in most cases replace the REPL. No amount of default editor hangover (I'm looking at you [IDLE](https://docs.python.org/3/library/idle.html)) can save you there.

By way of a counter-argument, sometimes the product can be so good, more importantly the marketing around the *new paradigm* can be so convincing that this is the new world you should be getting behind, that users actively switch and forget to ask for better air or gravity. The one example I remember is the iPhone which famously didn't have cut and paste in the early days - AND IT DIDN'T KILL THE PRODUCT. It was the Apple of then - which was an extension of Jobs - that could convince users they were on Mars and not shitty Earth, and silently add the feature in later on. The Apple of now tries to imitate what they had, with pulling jacks out of phones and removing all but a single type of port from their laptops, but it rings hollow. It isn't sold as well, it isn't understood as well by the people doing it, and so when they say **COURAGE** on stage people look at it like the youtube ads that say **KNOWLEDGE**.

(Venting the little rant I keep bottled up on that topic: removing ports to push people into the new age isn't how technology moves forward. Any new standard has the possiblity that massive use will expose problems that the designers couldn't conceivably think of. We lived with DVI while we figured out HDMI1 and then 2. There is no substitute for actual use at scale - no amount of testing or standards committees will solve that. THAT'S why we don't do that and the clusterfuck around USB-C connectors and protocols - is it displayPort over USB-C or Displayport over USB-C or DisplayPORT over USB-C - proves that. Also just the fact that you announce your company as falling behind a one port philosophy and forcing your users to convert when your mobile lineup and laptop lineup use TWO different one port solutions that have different licensing, power and hardware configurations shows it isn't really coherent. Sorry about that.)

On the flip side of this story, this kind of new gravity also creates ecosystems that engender zealots who will fight for you once they've been initiated. Vi and Emacs come to mind. Don't get me wrong, I like EMACS. I like LISP, and I've lived in EMACS for a long time. I might still go back once I'm retired, but I left once I wanted to stop treating it like an operating system. I realised I had to - if your air smells like EMACS and your gravity feels like EMACS it feels weird to be anywhere else. I was looking at every system in life and wondering if I could whip up some code to do it in the editor so I didn't leave. Can I do email? Task Management? Phone-calls from Mum? Eventually I gave up while wondering what it would be like to improve the internal scheduler so I could load more complex programs. Vi is worse in our way or the highway. So I now work in nano - I know, I'll give you a second to leave disgusted that I've suckered you into believing that I know anything. 

If you haven't, hear me out. It's available everywhere, it's light as hell, and it does what I need it to. When I open up data on a terminal, most of the time I'm looking to make a quick edit, save and get the hell out. Nano is near perfect. It's an editor that - and this is revolutionary - opens in EDIT mode, shows me the data, and has commonly used shortcuts for things right at the bottom because of course I forget that one thing I haven't used it forever. Of course I can get my emacs pinky out and google the right meta-thing for my function, and add it to my config file with a handy shortcut which I'm also now at risk of forgetting if I don't use it for another five years. I got tired of carrying around my .emacs file.

But until I moved out, I was willing to fight tooth and nail for my EMACS world. I understand the people that fight for Apple, VIM and Eclipse, I do. Except I don't think anybody fights for Eclipse.