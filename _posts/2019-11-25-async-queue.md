---
layout: post
title: "Async Interval Queue"
date: 2019-11-24 20:16:08
image: 'AsyncQueue/locks.jpeg'
share_image: 'AsyncQueue/locks.jpeg'
description: "A timed queue for javascript Promises"
tags:
- Async
- Promises
- Javascript
- Queue
- Rate limiting
categories:
- Software Builds
twitter_text:
---

Some of the most used projects I've built have been things I wrote for myself, and some of the best pieces of code I've used have the same origin. So when I look for something to fill a need and I'm unable to find it, I try and put it back out there when I end up writing one for myself. Chances are, someone else will need one at some point, and I'm happy if my work can serve as a starting point - at worst case an example of what not to do - so we all collectively move forward.

![wisdom]({{site.url}}/assets/img/AsyncQueue/wisdom_of_the_ancients.png)

[Async Interval Queue](https://www.npmjs.com/package/async-interval-queue) is a simple queue (50 lines of code, no dependencies) for Javascript that lets you enqueue a job and run it on a timer, while the rest of your code is unaware.

You simply import it, create a new queue with an interval, then add [thunked](https://en.wikipedia.org/wiki/Thunk) jobs in the same place your jobs were meant to return:

```javascript
const AsyncQueue = require('async-interval-queue');

// Create a new queue with running interval in ms
let myQueue = new AsyncQueue(1000);

// Let's make an async function for tests
async function myFunc(value) {
  return value;
}

myQueue.add(() => Promise.resolve("Hello")).then(console.log);
myQueue.add(() => Promise.resolve("World")).then(console.log);
```

Or, if you'd prefer - you can use the decorator to wrap the function so that you don't need to thunk everything.

```javascript
let myQueuedFunc = myQueue.decorator(myFunc);

myQueuedFunc("from").then(console.log);
myQueuedFunc("Decorators!").then(console.log);
```

The queue stops when all jobs are done, and new jobs will start the queue if it isn't running. There's an additional parameter to keep jobs from starting the queue, and manual start and stop functions. The [README](https://github.com/hrishioa/async-interval-queue#readme)and [tests](https://github.com/hrishioa/async-interval-queue/blob/master/test.js) cover it in a little more detail.

But that's it. My use was to limit API calls to a certain rate, and while I could find packages that did this on the server side (when you're the API that needs limiting), I couldn't find a good simple way to do this when you are the caller.

Feel free to use the package, or just the code - it's a small single file. We're using this in production at [Greywing](https://grey-wing.com), so any feedback is appreciated.

## Next Steps

The queue is completely in-memory, which makes sense since the jobs aren't easily serializable. What this makes complex though is running jobs in a cluster or multi-threaded environment. That's the next step for me. I've got an implementation in my head, and once I've got it up and running I'll post it here.