---
layout: post
title: "Debugging Node.js in Production: Anatomy of a bug hunt"
date: 2019-02-02 02:51:03
image: 'DebugNode/cover.jpeg'
description:
tags:
categories:
twitter_text:
---


# Background

One of last weekend's projects was building a messenger bot. I'd tried to build one before, but never really started coding as none of the projects I had in mind really worked well with the format of a chatbot. As with blockchain or any emerging technology, trying to shove a new piece of tech into something when it doesn't fit isn't a pleasent experience. So I'd left it for a while, and I had built the backend of some bots recently - [Trainify](https://m.me/608739242899742) and [Power Uncle](https://m.me/powerunclesg) (both of which I might write about at some point - Trainify got 3000 users in two days but it raised more problems than it solved). I built something else called WifePoints (really really weird story there, but let's just say it's currently just internal use). I thought I'd give it another shot, and started on Zork.

![zork screenshot]({{site.url}}/assets/img/DebugNode/zork-banner.jpg)

[Zork](https://en.wikipedia.org/wiki/Zork) - which might be familiar if you were gaming before TTYs went away as the default user interface - was a brainchild of the 70s, when non-learning NLP was starting to take off through projects like itself and [ELIZA](https://en.wikipedia.org/wiki/ELIZA) before it. With incredibly light and decently good text processing for its time, Zork (and later Zork II and III) were interactive text worlds with monsters, dark rooms, complex puzzles and amazing adventures. If you've never played one, it was somewhat like playing Dungeons and Dragons over text with a smart but pedantic dungeon master. If you've never played one of those, it was exactly like Fortnite.

Having been a fan of the game, it seemed like a good fit for a messenger bot. The interface was designed for text, small responses and immersive storytelling with light NLP. So while it didn't turn out as easy I'd expected it to, I ended up building one for Messenger.

![zork messenger screenshot]({{site.url}}/assets/img/DebugNode/threed_mockup.png)

# Build

I'll give a quick overview of the build and the tech stack here, with some of the challenges therein. I initially expected this to be easy. With a game as popular as Zork and as old I expected to find source code I could port overand run. But it turns out the best I could find for source was [the original source code](https://github.com/devshane/zork) written for the [PDP-10](https://en.wikipedia.org/wiki/PDP-10), which if you've never heard of it is a wire-wrapped tape-run beast of a machine, with a 36-bit word length and 1 microsecond cycle time (1 microsecond really) that looked like this:

![zork messenger screenshot]({{site.url}}/assets/img/DebugNode/PDP10.jpg)

The source code I found was thankfully ported from its original FORTRAN to C, but it was quickly clear that getting this up and running was already a weekend project in itself. And did I say ported? I meant translated. This is GOTO style C code that is a lot closer to assembly than modern C - I'll take it any day over Android disassembly, but not exactly a dream.

```c
L5000:
    more_output(NULL);
    printf("V%1d.%1d%c\n", vers_1.vmaj, vers_1.vmin, vers_1.vedit);
    play_1.telflg = TRUE_;
    return ret_val;

/* V75--	SWIM.  ALWAYS A JOKE. */

L6000:
    i = 330;
/* 						!ASSUME WATER. */
    if ((rooms_1.rflag[play_1.here - 1] & RWATER + RFILL) == 
	    0) {
	i = rnd_(3) + 331;
    }
    rspeak_(i);
    return ret_val;
```

Moving on, I found two ports, the [first](https://github.com/bburns/Lantern/blob/master/src/lantern.py) from [Muddle](https://en.wikipedia.org/wiki/MDL_(programming_language)), which I had to look up. Turned out to be a LISP dialect (oh one of those, let me just get my trust LISP interpreter out - wait which one), and the other was a half-completed python port that barely said hi. ZORK is available on emulators online, so I tried accessing those, but it turns out most of them run full emulator binaries with the image loaded into your browser (what a time to be alive, really)! Finally, just as I was about to declare defeat, it turned out that Infocom has open-sourced [Frotz](https://davidgriffith.gitlab.io/frotz/), a [Z-Machine](https://en.wikipedia.org/wiki/Z-machine) interpreter - also from the 70s, made for and partly named after Zork. Z-Machine is a virtual machine for text-based games, with its own instruction set - I guess before DSLs we had Domain Specific Machines. Frotz has some support available, and to my delight - two lightly (and that's being generous) maintained interface packages for Node, which look suspiciously like [each](https://www.npmjs.com/package/frotz-interfacer) [other](https://www.npmjs.com/package/node-frotz). What's more, they seem to be from different developers but the quick start code for one uses the other package instead of itself. Never a dull day in Nodeland.

The system was not without its benefits though. Each time a call to Zork is made, the package spins up a new instance of frotz through a child process, loads a save, runs the input, returns the output, saves the game and exits. Ordinarily this would be extremely wasteful for a continuous game - and it is, but in this case it was a rather large feature and not a bug for me. This meant that the calls could be truly stateless with local persistence built in, and if I managed saves per user, I wouldn't really have to worry about scaling or intermediate termination and concurrency issues.

So I implemented it into a simple Node program that basically connected a [Bootbot](https://github.com/Charca/bootbot) instance (beautifully done package for creating bots, only problem is with multiple workers) with the package and some basic user management and analytics, and I was done. The real operating core of the program was less than 200 lines, and I was happy. The original code (all 178 lines of it) is [here](https://github.com/hrishioa/zorkbot/blob/1e17c3abf2cdb9dc1a0fbd34106efc7b5c07adcc/app.js), but I wouldn't recommend you run it as is, for reasons that will be clearer soon.

All was good. I had some friends test it out, and no problems really emerged. I decided I'd leave scaling for later (if ever), called it a night and moved on to polishing. Decent output for a few hours of work, I had support for all three versions of Zork, some save management and everything seemed good.

# Problems

![grue]({{site.url}}/assets/img/DebugNode/grue.jpg)

As I used it for a day or two, I noticed that the bot would stop responding when talking to Zork every now and then. Seemed like a standard dropped connection. I knew that Facebook would sometimes wait a while, then bundle responses and send them over all at once so I presumed it was the bootbot instance either **a**. trying to send multiple commands to Zork, or **b**. sending them individually and the frotz instances were having problems taking a lock (or a goddamn mutex for all I knew) for the same savefile at the same time. I thought it might be an easy fix with a delay or some checking, left it for later. Part of me wondered if it was a memory issue with the small cloud instance I was using for a server. All put, I kind of lied in the title - this was production in that the bot was released, but it was far from a production-grade stack. I was using pm2 to cluster manage instances of node that were sitting behind a free pipe to ngrok (so I could get over SSL restrictions from a naked IP), and running barely supported npm packages that weren't even tested for my current version of Node. In fact, Bootbot liked one version and frotz-interfacer liked another. I ended up shafting the more pliable of the two - which turned out to be frotz-interfacer - and going with Bootbot's preference. So yeah, largely a house of sticks built on a soap bubble. I deserve my life.

Then came Friday, and I had somewhat of a surprise looking at the user retention. The bot had been passed around a fair bit, and users were actually interacting and coming back to talk to the bot. The average user had already exchanged over 700 messages, and my girlfriend seemed to be completing missions left and right (I still suspect she was looking up walkthroughs, but I can't prove it - yet). I should probably fix that bug.

I looked at the logs. This wasn't a bug that was easy to reproduce to begin with, in that it occurred randomly while you were talking to the bot. Looking at the error logs, it seemed that whenever it occurred, the node process spit out the following message and promptly died. No crash report, nothing else. Just sudden death.

```bash
events.js:72
        throw er; // Unhandled 'error' event
              ^
Error: read ECONNRESET
    at errnoException (net.js:901:11)
    at Pipe.onread (net.js:556:19)
e
```

That e at the end isn't a typo - sometimes the process would die in the middle of printing the error message. It never really got past the e, leaving me wondering in suspense as to what my dying process was trying to tell me before it had its throat slit by (presumably) the OS. BTW the little comments from the source code made things worse. Yes, I know it's an unhandled error event, but what on earth does 'alternatively it s a write' mean? There's absolutely no information here as to who is failing, why or when. Googling the problem produced mostly other confused conders, but none of them used any of the packages I was using - I only really had two dependencies, and even their dependencies didn't match the other people this error frustrated.

I was still pretty lazy, so I tried improving exception handling in my code (which through years of coding I'm glad to say was the only thing not terrible about this codebase at the time) to catch any errors that were falling through. Most everything was wrapped in promise code that caught errors, but I added catch-alls just in case and waited. Nothing. Nada. The same error message, and none of my handlers had triggered.

```bash
events.js:72
        throw er; // Unhandled 'error' event
              ^
Error: read ECONNRESET
    at errnoException (net.js:901:11)
    at Pipe.onread (net.js:556:19)
e
```

I tried improving logging - I was getting less lazy and more intrigued so I figured I'd use [Winston](https://www.npmjs.com/package/winston) like I should have from the beginning. Unfortunately Winston's not very good at logging objects, and it doesn't seem like it would solve the problem - my handlers weren't really triggering in the first place. [Longjohn](https://github.com/mattinsler/longjohn), with the promise of async help didn't really pan out either. I got desperate enough to mess with `console.log` (wait - isn't that how we all debug?) using [console-stamp](https://www.npmjs.com/package/console-stamp). It's definitely printing to console, maybe that'll help. 

Nothing.

```bash
events.js:72
        throw er; // Unhandled 'error' event
              ^
Error: read ECONNRESET
    at errnoException (net.js:901:11)
    at Pipe.onread (net.js:556:19)
e
```

All of my console messages were now neatly stamped and color coded, but any crashes still had the exact same message. I triple-checked that was running the code I was editing, because nothing I did seemed to have any effect. My new guess was that this must have something to do with the web part of things, due to that net.js result. Quickly turning off frotz-interfacer and replacing it with a dummy echo line seemed to fix the problem though, so that must be where it is. This turned out to be a blessing in disguise, because Bootbot while way more supported had a lot more known bugs and a lot more code to dig through.

`frotz-interfacer` on the other hand, had one [file](https://github.com/jwoos/javascript_frotz/blob/master/lib/index.js). Well, two files but the other was really just an error file that looked like this - 

```javascript
'use strict';

class FileError extends Error {
	constructor(message) {
		super(message);

		this.name = 'FileError';
	}
}

class DFrotzInterfaceError extends Error {
	constructor(message) {
		super(message);

		this.name = 'DFrotzInterfaceError';
	}
}

module.exports = {
	FileError: FileError,
	DFrotzInterfaceError: DFrotzInterfaceError
};
```

- so we're looking at `index.js`. This was honestly the first time I was looking at the source repo for the package, so it didn't really inspire any more confidence in my success when I noticed that the last commit was 3 years ago with a lot more open issues than I expected and no pull requests. Still, the code wasn't al that bad. Quite simply, the package takes an input and performs the steps I outlined above - start game with save, run input, get output, save new game, exit - using node's own [child_process]. This really seemed far too simple to be problematic.

# Diagnosis

The most complicated function - and my entry point to the package - was `iteration`. So I did what any of us would do. I peppered it with `console.log` (I later did what none of us should, which was to commit it to a public repository - ruining any chance I ever had at working for a living).


```javascript
function iteration(cmd, cb) {
	var _this4 = this;

	var save = void 0;

	return this.checkForSaveFile().then(function (saveFileExists) {
		console.log("Frotz - savefile exists - ",saveFileExists)
		_this4.dropAll = saveFileExists;
		save = saveFileExists;
		_this4.init(cb);
		return bluebird.resolve(_this4.dfrotz);
	}).then(function () {
		console.log("Frotz - Initing...")
		return _this4.restoreSave(save);
	}).then(function () {
		console.log("Frotz - delaying...")
		return bluebird.delay(100);
	}).then(function () {
		console.log("Frotz - Starting run - command is ",cmd)
		_this4.dropAll = false;
		if (cmd) {
			_this4.command(cmd);
		} else {
			throw new errors.DFrotzInterfaceError('A command must be provided');
		}
		console.log("Frotz - Delaying again...")
		return bluebird.delay(100);
	}).then(function () {
		console.log("Frotz - Writing save...")
		try {
			return _this4.writeSave();
		} catch(e) {
			console.error("DFrotz error saving - ",e)
			return bluebird.resolve()
		}
	}).then(function () {
		console.log("Frotz - Wrote save.")
	}).catch(function (e) {
		// do something
		console.error("DFrotz Error - ",e);
		throw e;
	});
}
```

I added some exception handlers of my own too, at least now I could see what was going on behind the scenes.

```bash
[[14:50:29.007]] [LOG]    Frotz - Initing...
[[14:50:29.018]] [LOG]    Frotz - delaying...
[[14:50:29.121]] [LOG]    Frotz - Delaying again...
[[14:50:29.222]] [LOG]    Frotz - Writing save...
Error: read ECONNRESET
    at exports._errnoException (util.js:1020:11)
    at Pipe.onread (net.js:580:26)
[[14:50:29.241]] [LOG]    Frotz - Wrote save.
```

At this point I was more confused than ever. It seems to be an async operation that's throwing the error - possibly the child process. However even though it looks like it, I couldn't be sure that it was the save operation that was causing the error. Also, I'd forgive you if you were so tired of seeing the error so many times that you'd missed it this time. Look again.

The goddamn line number for the error had changed! Despite me not changing anything in util.js or net.js (talk about generic repeated names), somehow I was throwing an error from somewhere else. Thankfully I didn't need to investigate this too far, because this is where things started looking up. Some of my old training in Node kicked in and I restarted the process and errored it again. Nothing is certain in Nodeland until it's the same about 70 times out of a hundred runs. 

```bash
[[15:03:26.164]] [LOG]    Frotz - Initing...
[[15:03:26.179]] [LOG]    Frotz - delaying...
[[15:03:26.382]] [LOG]    Frotz - Writing save...
Error: read ECONNRESET
    at exports._errnoException (util.js:1020:11)
    at Pipe.onread (net.js:580:26)
---------------------------------------------
    at Socket.Readable.on (_stream_readable.js:691:35)
    at /root/zork/node_modules/frotz-interfacer/dist/index.js:168:12
[[15:03:26.407]] [LOG]    Frotz - Wrote save.
```

I was honestly too happy at this point to care why. Somehow my logging extensions had kicked in - maybe a precompiled cache had refreshed, maybe the node gods wanted me to finish watching my movie I'd paused (quick bugfix, why not). I was on the right track, and I had some more info on the error as long as I didn't run it again and get something else. 

Zooming in further, the error seemed to be coming from the childprocess part of the code (as suspected), which was the command function. So I added some error handlers there, see if we can trap this apparition once and for all.

```javascript
function command(cmd) {
	console.log("Frotz - running command ",cmd)
	var _this = this;

	var timeout = arguments.length <= 1 || arguments[1] === undefined ? 10 : arguments[1];

	var promise = new bluebird(function (resolve) {
		console.log("Frotz - actually running command ",cmd)
		try {
			_this.dfrotz.stdin.write(cmd + '\n', function () {
				resolve();
			});
		} catch(e) {
			console.error("Frotz - error running command - ",e)
			resolve()
		}
	});

	return timeout ? promise.delay(timeout) : promise;
}
```

Nothing. Bupkus. Same message, none of them trigger. At this point I'm starting to give up and beginning to consider writing a fork process that repeats an action if a certain error message pops up on console for redundancy and thereby resigning myself to never be able to call myself a programmer ever again with any modicum of self-respect, when I try googling the error again, this time with childprocess.

![childprocess error]({{site.url}}/assets/img/DebugNode/childprocess.png)

Ah! Turns out this is a known issue in Node's childprocess (which despite being standard Node issue and rolled out as part of the standard environment hasn't been fixed yet - the friggin issue is closed). If you don't register an error handler for your child process WHEN YOU'RE CREATING IT the process will burp to console and crash everything, including the parent. Well why didn't you say so?

```javascript
function init(cb) {
	var _this3 = this;

	var output = '';
	var dfrotzArgs = ['-w', '500', '-h', '999', this.gameImage];

	this.dfrotz = childProcess.execFile(this.executable, dfrotzArgs, { encoding: 'utf8' }, function (error, stdout, stderr) {
		if (!stderr && !error) {
			output = output.replace('\r', '').split('\n');

			if (_this3.outputFilter) {
				output = output.filter(_this3.outputFilter);
				output[0] = DFrotzInterface.stripWhiteSpace(output[0], true);
			}
		}

		cb({
			stderr: stderr,
			error: error
		}, {
			pretty: output,
			full: stdout
		});
	});

	this.dfrotz.stdin.on("error", function (e)
	{
	    console.log("STDIN ON ERROR");
	    console.log(e);
	});

...
```

- and, we get this.

```bash
[[15:07:03.524]] [LOG]    Frotz - Initing...
[[15:07:03.525]] [LOG]    Frotz - running command  restore
[[15:07:03.526]] [LOG]    Frotz - actually running command  restore
[[15:07:03.527]] [LOG]    Frotz - running command  ./saves/zork1/2295256887161761.dat
[[15:07:03.527]] [LOG]    Frotz - actually running command  ./saves/zork1/2295256887161761.dat
[[15:07:03.540]] [LOG]    Frotz - delaying...
[[15:07:03.641]] [LOG]    Frotz - Starting run - command is  Go south
[[15:07:03.642]] [LOG]    Frotz - running command  Go south
[[15:07:03.642]] [LOG]    Frotz - actually running command  Go south
[[15:07:03.646]] [LOG]    Frotz - Delaying again...
[[15:07:03.747]] [LOG]    Frotz - Writing save...
[[15:07:03.748]] [LOG]    Frotz - running command  save
[[15:07:03.748]] [LOG]    Frotz - actually running command  save
[[15:07:03.750]] [LOG]    Frotz - running command  ./saves/zork1/2295256887161761.dat
[[15:07:03.751]] [LOG]    Frotz - actually running command  ./saves/zork1/2295256887161761.dat
[[15:07:03.752]] [LOG]    Frotz - running command  Y
[[15:07:03.752]] [LOG]    Frotz - actually running command  Y
[[15:07:03.755]] [LOG]    Frotz - running command  quit
[[15:07:03.756]] [LOG]    Frotz - actually running command  quit
[[15:07:03.760]] [LOG]    Frotz - running command  Y
[[15:07:03.761]] [LOG]    Frotz - actually running command  Y
[[15:07:03.761]] [LOG]    Frotz - running command  SI
[[15:07:03.761]] [LOG]    Frotz - actually running command  SI
[[15:07:03.769]] [LOG]    STDIN ON ERROR
[[15:07:03.770]] [LOG]    { Error: read ECONNRESET
    at exports._errnoException (util.js:1020:11)
    at Pipe.onread (net.js:580:26)
---------------------------------------------
    at Socket.Readable.on (_stream_readable.js:691:35)
    at DFrotzInterface.init (/root/zork/node_modules/frotz-interfacer/dist/index.js:136:22)
    at /root/zork/node_modules/frotz-interfacer/dist/index.js:168:12
    at tryCatcher (/root/zork/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/root/zork/node_modules/bluebird/js/release/promise.js:512:31)
    at Promise._settlePromise (/root/zork/node_modules/bluebird/js/release/promise.js:569:18)
    at Promise._settlePromise0 (/root/zork/node_modules/bluebird/js/release/promise.js:614:10)
    at Promise._settlePromises (/root/zork/node_modules/bluebird/js/release/promise.js:694:18)
    at _drainQueueStep (/root/zork/node_modules/bluebird/js/release/async.js:138:12)
    at _drainQueue (/root/zork/node_modules/bluebird/js/release/async.js:131:9)
    at Async._drainQueues (/root/zork/node_modules/bluebird/js/release/async.js:147:5)
    at Immediate.Async.drainQueues (/root/zork/node_modules/bluebird/js/release/async.js:17:14)
    at runCallback (timers.js:672:20)
    at tryOnImmediate (timers.js:645:5)
    at processImmediate [as _immediateCallback] (timers.js:617:5) code: 'ECONNRESET', errno: 'ECONNRESET', syscall: 'read' }
[[15:07:03.775]] [LOG]    runGame - interfacer returned.
[[15:07:03.775]] [LOG]    runGame returning  [ 'South of House | Score: 0 | Moves: 9',
  'South of House',
  'You are facing the south side of a white house. There is no door here, and all the windows are boarded.' ]
[[15:07:03.775]] [LOG]    Talk - runGame returned...
[[15:07:03.781]] [LOG]    Frotz - Wrote save.
```

I shortened the story a fair bit. There was a lot more hair pulling and logging and digging into functions than that, but at this point the error is caught. Node behaves as though the error doesn't exist, which is fine with me - in all my tests, the game runs and saves perfectly and would be working ideally if it wasn't for Node throwing a hissy fit and crashing. Which it doesn't anymore. All good.

# What did we learn?

Not much really. I remain the same doof as I was before, with one less bug in my life. None of this was really ripping on Node though - even though it was. All of the things that made me choose this stack - availability of somewhat decent code for even obscure things, easy and short code creation, installation and scaling - also make it a hotbed for badly written and maintained code both on my side and of the code I'm borrowing. And now that I've fixed the one bug I see, I'll probably go on and add a README to the repo and move on with my life, and there it will lie. Until that day maybe years into the future when someone else will go 'I should build a skynet bot using an old messenger bot!' and you'll see another one of these posts.