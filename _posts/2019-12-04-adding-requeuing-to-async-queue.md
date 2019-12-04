---
layout: post
title: "Adding Requeuing to Async Interval Queue"
date: 2019-12-01 21:58:06
image: 'AsyncQueue/requeue.jpg'
share_image: 'AsyncQueue/requeue.jpg'
description: "Working with promises"
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

As a quick addition to [Async Interval Queue](/async-queue), I've also added the ability for the queue to requeue failed jobs and run them again, and this took some JS-wrangling as well. The old code for enqueuing a job looked like this:

```javascript
add(asyncFunction, doNotStart) {
  return new Promise((resolve, reject) => {
    this.queue.push(() => {
      asyncFunction().then(value => resolve(value)).catch(err => reject(err));
    });
    if(!doNotStart && !this.interval)
      this.start();
  })
}
```

> This is one of those instances when the unstructured mess that is Javascript can be made to work in your favor. What it also means if that you don't know if you're creating more bugs than you're solving, and just delaying the introduction of those bugs way into the future.

You returned a promise that pushed a [thunk](https://en.wikipedia.org/wiki/Thunk) onto the queue array, which when ran would execute the function and resolve the promise when that function completed. Due to the magic of JS namespaces and closures (or how fast and loose JS plays with these concepts), this works. I would have a terrible time getting this to work in Java, but that is *not* me saying that Java is a better language. If anyone wants to give this a try on Haskell or Rust, I would be very excited.

To implement requeuing, we modify the add function to be a bit more complex:

```javascript
add(asyncFunction, doNotStart, retries) {
  let thisQueue = this;
  return new Promise((resolve, reject) => {
    let job = {
      task: () => {
        asyncFunction().then(value => resolve(value)).catch(err => reject(err));
      }
    };

    if(retries && retries !== 0) {
      job = {
        task: () => {
          asyncFunction().then(value => resolve(value)).catch(err => {
            return thisQueue.add(asyncFunction, doNotStart, retries-1).then(val => resolve(val)).catch(err => reject(err));
          });
        }
      }
    }

    this.queue.push(job);
    if(!doNotStart && !this.interval)
      this.start();
  })
}
```

We accept a parameter that specifies the number of retries, then recursively (but in a lazy fashion) requeue the same job until we have a successful run or we've run out of retries.
The key part (where we requeue the job) is this one:

```javascript
asyncFunction()
  .then(value => resolve(value))
  .catch(err => {
    return thisQueue.add(asyncFunction, doNotStart, retries-1)
              .then(val => resolve(val))
              .catch(err => reject(err));
  }
);
```

Now if you have a network request that might fail, you can ask for a retry rather than having it fail right away.

Due to the magic of closures, we can keep the original asyncfunction in the namespace and add it back to the queue, propagating the resolve or reject back up as the promises unwind. I've done my best to keep from having too many chained promises. Javascript's behavior with chained promises and await is (to me) still somewhat random. Sometimes they're unpacked, sometimes they're not. I hope to god that the GC is all right with this and doesn't leak memory, but I haven't tested this yet.

Testing this functionality is simple enough. We create a function that will succeed or fail with some probability:

```javascript
function probabilistic(value, pSuccess) {
  return new Promise((resolve, reject) => {
    if(Math.random() < pSuccess) {
      resolve(value+" succeeded");
    } else
      reject(value+" Rejected due to probability");
  })
}
```

We then queue this function with some retries to see if the expected behavior happens:

```javascript
let myQueue2 = new AsyncQueue(1000);
let pFunc = myQueue2.decorator(probabilistic, false, 2);

pFunc("one",pSuccess)
  .then(val => console.log("one succeeded with ",val))
  .catch(val => {console.log("one failed with ",val))
pFunc("two",pSuccess)
  .then(val => console.log("two succeeded with ",val))
  .catch(val => {console.log("two failed with ",val))
pFunc("three",pSuccess)
  .then(val => console.log("three succeeded with ",val))
  .catch(val => {console.log("three failed with ",val))
```

The tests were successful (I played around with different number of retries and probability of success), and I've added it to v1.0.7 of [async-interval-queue](https://www.npmjs.com/package/async-interval-queue). The complete [test code](https://github.com/hrishioa/async-interval-queue/blob/master/test.js) is availabile to try out, and will show the completed order of execution and report any retries.

Overall, I think I'll pause work on the async queue for now. What I'd really like is a version that supports parallelism, and working on this one much further might be putting that on hold.