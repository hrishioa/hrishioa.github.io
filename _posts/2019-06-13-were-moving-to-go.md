---
layout: post
title: "We're moving to Go"
date: 2019-06-12 18:31:06
image: 'MovingToGo/cover.jpg'
share_image: 'MovingToGo/cover.jpg'
description: "Sorry for ranting I'm getting older"
tags:
- Go
categories:
- Rants
twitter_text:
---

This post was inspired by a conversation I had a few days ago, where a friend of mine told me about the recent rewrite their systems were undergoing so they could move from a node backend to Go. I didn't finish that conversation. I'm going to, but I wanted to think and talk about it first. I didn't want to really ask what was going on, because I wanted plausible deniability that his case didn't belong in here (What? No! There's spousal abuse in other countries, its a real problem, but no one I know does it or I'd have to intervene). For another, the structure of the conversation led me down the PTSD-laden rabbit hole of new languages and processes. 

## Don't get me wrong, Go is awesome

It really is. Multi-process communication, better typing - honestly I don't really need to go into detail about why Go as a language spec is significantly better than the [dumpster fire](http://hrishioa.github.io/debugging-node-in-production-anatomy-of-a-bug-hunt/) of duct tape that is modern Javascript. 

So what's the problem? The problem is changing languages. It's very, VERY rare that the benefits outweigh the risks. Let me try and defend that.

There's something that most developers with some experience will know and agree with.

## Maintaining code is hard, writing new code is easy

Every developer I've known has had that moment of looking at legacy code where they wanted to rm -rf the entire thing and start writing again. Legacy code in this context means one of two things - it's old, or it belongs to someone else. If it's both its definitely legacy code and you're going to need superhuman control not to chuck it in the bin, especially if no one's looking. Functionally, if it's new to your mind it's legacy code. This is something I've encountered and given in to many times in my life, and something I've obsessed over fixing many times. If you'll permit a brief interlude, let's consider why this happens.

## The problem isn't the code, it's what isn't in it

Every developer I've known has also had that moment when they're building things where - for a brief moment, or longer depending on program complexity or the amount of coffee in your system - you see the matrix. You know the entire program, where it's been where it is and where it's going, you know the dependencies, you can probably trace program execution across libraries in your head. This is the runner's high a lot of us got into coding for. Problem is, this points to the fact that the code itself isn't enough. Very little in your head at that moment is code - everything else is metadata, information that isn't part of your however many lines of code. We've tried to segment and capture this information, tried to follow the first instinct we all have when we find a dream we don't want to lose - we *write it down*. This is how we get documentation, API specs, dependency locks, config files and product specs. We figured them out after years of struggle as a species, and they work - when done **absolutely** well. Except they rarely are, for the simple reason that they don't serve a strong purpose to the person doing them in the moment, and they don't contribute to actual development or progress, in the moment. You **stop** developing to do these things. 

Regardless of how far we've evolved, our brains are yet to internalise amortization and fold in future costs. We know the world is going to get hotter and harder to live in, but we want the flights we want today. Someone once told me that to be good at something takes good instinct, and good instinct comes from bad experience. The same way, a developer with good instincts built over hard years knows that trying to fold this metadata onto paper will save way more time and money than you lose in the moment. It won't look as elegant as it did in your head - it will be filled with redundancies, some of the neat connections won't be readily apparent, and you know the next guy won't be enthused to read it. But it's the best we've got and it keeps the madness a little bit further. Still, every now and then we'll invent self-documenting code and rejoice that all we need to do is code now, and forget that the little function signatures with return types were only 0.01% of the entire thing.

It's a hard thing to ask our best and brightest programmers to do, but they're the only ones who can do it. You really can't ask someone else to write this documentation for you - they don't know what you know. If you could communicate it to them, that **is** documentation. Combine this with the fact that writing new code (you get to think about architecture! imagine how much bloat you'll eliminate!) is incredibly fun, and it's a miracle we get ourselves to do it at all.

Changing languages is often motivated by the same things as changing codebases. Old languages look bad for the same reason old code does. It's clunky and big, definitely not as fast as you KNOW this program should be running. It's covered in tape and hotfixes, the community is split over this and that and sides alternate their victories so there's no core philosophy anymore, but look at this shiny new thing! It's full of potential and no negative history! Its definitely learned things from the ones that came before - or at least it claims to - and but the most important thing is it's not fuck ugly. It's got better benchmarks so you know it'll be faster. Plus, you get to rewrite things from scratch.

## The tape is there for a reason

Problem is, likely the old language started out the same way. It was young and thin and naive once. It looks scarred and burdened with the weight of a thousand pull requests, but they're there for a reason. Much like the old code, some of them are there for execution reasons, some for compatibility, and a lot for things that a new language cannot foresee or implement in its nascency. We live in a world with different people, different computers, different hypervisors. The edge cases are endless and the [RAID is huge with bugs](http://www.workpump.com/bugcount/bugcount.html). When you throw that old monstrosity in the bin, you forget the screams and [the wisdom of the ancients](https://xkcd.com/979/), and you resign yourself to the same fate. Your best case then is that you can pass that onto another unsuspecting dev before you hit them.

## Learning a new language is hard

Among the people I've known (and become), junior devs are the most likely to want a new language. Senior devs (and god forbid sysadmins) would often rather burn the entire building than change the stack to something cool and new. They'll rest happy in their little cell in prison, knowing it was better than the alternative. This is (at least in part) because early in your career, you think **learning a new language is easy**. "Oh Go, I can pick that up in a weekend", you think to yourself. You're right, but only a little.

I realise I'm speaking for people that can't defend themselves, so let me use myself as an example. As of right now, with what I know and the internet, it will take me 2-4 hours to start coding in language X. By that I mean it'll take me that long to get proficient at pasting stackoverflow code into a file. It'll run, and depending on the size it may even be readable - and by a lot of standards it should now go in my resume - but am I a language X programmer yet?

Give me a weekend and I'm better. I now understand bits of the language, and I'm not online as often. I can probably do a fizzbuzz without Googling (I'm still proud I can write a tar command without Googling so I should probably calm down), and I can write modular code that is - depending on the size - readable to another person. Give me two senior developers who spend twice as long on code review than I do coding, and I can write production code. Am I a language X programmer yet?

It's not a yes, it's not a no, it's just better than before, and this is the problem. It took me a long time to humble myself and realise that it will take months of writing different kinds of code before I understand the intricacies and idiosyncracies of the new idiom I chose to adopt. What the **language X** way of doing things should be. I know which memory leaks to watch out for, what kinds of logging to install because those errors **will happen*,*** what things need to be simple and what can take complexity. You only find them when you fail - you might get halfway there if you're lucky and someone writes an [what AWK has](https://ia802309.us.archive.org/25/items/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) for you (if you haven't read it please do - best favor you'll do yourself this week). Usually, there's none to be found, and you don't really know how to look for it on stackoverflow. When you do find someone talking about it, there's always multiple sides and strong flamewars, and you don't know enough so you nod along and go with the most persuasive argument in your ignorance.

## The final match

This is a pretty dangerous cocktail, but even pretending that the world works the way I think it does (I'm usually content in the knowledge that it doesn't), things might still be fine. Junior devs are impulsive and exploratory, senior devs wish they could go back to wire wrapping and the conflict can have mutual benefits. However, the final thing that usually douses the entire thing in gasoline and strikes a match is the ever hated, ever necessary you-know-who. Oh goddammit I'll say the name: **management.** Management wants what junior developers want. Junior developers want the new thing and are willing to use the rose-colored glasses to write what is effectively ad copy which management from other firms read and hand out in commandments to senior devs. When this happens, if senior devs aren't willing to displease everyone with their displeasure, bad things often happen.

I'll say it again though: changing languages isn't a bad thing. It's changing them fast and without concern for why things are the way they are, and again - way too bloody fast - is what's bad. 

Combine this with the fact that the younger a language is (and I don't mean from inception, I mean from virality), the higher the percentage of developers you'll have that are just as young in their learning. Put enough of them in a room with a new language and a new project, and you've just as well as installed the 1000-ton roadblocks ahead of you. You just can't see them yet because they're far away, but they're there just the same and waiting. the best you can hope for is that when you hit it, you won't have a room full of people that'll stand up and shout 'this would be so much easier if we used language Y!'

## Addendum - one example of (the many) when I'm wrong

Of course I'm wrong. The world isn't black and white, and the advantage of writing opinionated commentary on the internet is that you speak to an audience that agrees with you. We remember the brightest points of our life, be it negative or positive. But of course I'm wrong - sometimes. Sometimes the system deserves to be ripped out and sacrificed to the gods and the site of its operation shut off from the world in a concrete sarcophagus like Chernobyl so others will know the horrors that happened here but they won't be exposed enough to get sick from it. 
Sometimes [Zawinski's Law](http://www.catb.org/jargon/html/Z/Zawinskis-Law.html) is true, and the duct tape is just tape and contains no wisdom. Sometimes the old language you're trying to replace was a bad choice for the job in the first place, or it's horrifically out of date with modern ecosystems and there's no point maintaining all this FORTRAN code.

What? The example, yes. The inimitable Avery Pennarun at [his blog](https://apenwarr.ca) talks about IPv6 by way of its problems, but all I could think about was [how badly IPv4 needs to go](https://apenwarr.ca/log/20170810). 