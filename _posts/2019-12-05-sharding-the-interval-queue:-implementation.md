---
layout: post
title: "Sharding the Interval Queue: Implementation"
date: 2019-12-05 00:45:27
image: 'ShardQueue/shards2.jpg'
share_image: 'ShardQueue/shards2.jpg'
description: "Promises, lists and setTimeout woes"
tags:
- Distributed Systems
- Sharding
- Async
- Promises
- Javascript
- Queue
- Rate limiting
categories:
- Software Builds
twitter_text:
---

The completed [Sharded Interval Queue](https://www.npmjs.com/package/sharded-interval-queue) is up on npm if you'd like to take a look or use it. This is the last of a two-parter on the theory and implementation.

In the [last post](/sharding-the-interval-queue-theory/) we looked at the theory on how we could implement an interval queue with sharding to support cluster workloads. Here we'll look at the implementation, design choices and pitfalls.

The [final code](https://github.com/hrishioa/sharded-interval-queue/blob/master/sharded-interval-queue.js) for the sharded-interval-queue is about 200 lines. Compared to 50 for the [async-interval-queue](https://github.com/hrishioa/async-interval-queue/blob/master/async-interval-queue.js), the rise in complexity was expected. What proved difficult was not adding more complexity to deal with issues that propped up.

## Shared state management

To start, let's ignore the exact implementation of the shared state - we'd like our final queue to be implementation agnostic. Building with a particular one in mind can inject platform-specific ways of thinking into our system and make the early testing and development more complex. For now, we'll use a global dictionary as a stand in:

```javascript
let globalKeys = {};
```

From our [first principles and constraints](/sharding-the-interval-queue-theory/#constraint-1), we know that we only need a few functions to operate on this shared state. We need to set and get keys specific to our queue and atomically increment them when possible - that's three. We'll add two helper functions - one that checks if a key is set, and another that initializes storage for when that's needed. We can represent those five operations like so:

```javascript
async isSet(key) {
  return (`${this.queueName}_${key}` in globalKeys);
}

async increment(key) {
  let val = globalKeys[`${this.queueName}_${key}`];
  globalKeys[`${this.queueName}_${key}`] = val+1;
  return val;
}

async setAsync(key, value) {
  globalKeys[`${this.queueName}_${key}`] = value;
}

async getAsync(key) {
  return globalKeys[`${this.queueName}_${key}`];
}

async initStorage() {
  globalKeys = {};
}
```

We're defining them as asynchronous functions, since we expect these operations to possibly go to disk or network for persistence. With these as a base, we've also made it easier to plug-in any custom shared state implementation into the queue: replace these five and you're good.

A queue name is used to associate keys, so you can have multiple queues running across shards, which connect to their brothers through a common name. `myQueue_key1` refers to key `key1` in queue `myQueue`. Simple enough to implement and inspect.

(As an aside, I now wish I'd made them aware of the queue name rather than pulling in from the class, as this would have allowed for custom implementations on how keys are associated with the queue name. As you'll see later I did make this work, but it's something I might change later.)

## Initialization

Next we build our constructor:

```javascript
constructor(queueName) {
  this.queueName = queueName;

  this.queue = [];
  this.index = 0;
  this.job = null;
}
```

However, we have a problem here that we didn't with the unsharded queue. Since our operations with shared state are asynchronous, we need to separate them from the constructor. Even though we can get all the relevant information when the object is created, constructors can't be asynchronous. Which means we need an `init` function:

```javascript
async init(intervalMs, reset, failIfExists) {
  await this.initStorage;
  if(!(await this.isSet('ticket')) || reset) {
    this.setAsync('ticket', 1);
    this.setAsync('current', 0);
    this.setAsync('interval', intervalMs);
    this.setAsync('paused', false);
  } else {
    if(failIfExists)
      throw new Error("Queue exists");
  }
}
```

Not much here but setup. We run `initStorage` to dust out whichver adapter we're using and create the queue variables if they don't exist. We also have parameters to reset the shared state for an existing queue (which is recommended but not necessary for multiple runs of the same code), and to throw an error if you expected to create a new queue but one exists.

## Adding jobs

Next we have the `add` function and the decorator. Thanks to our plan from before, this is still pretty simple:

```javascript
async add(asyncFunction, doNotStart) {
  let queueNo = await this.increment('ticket');
  let myPromise = new Promise((resolve, reject) => {
    this.queue.push({
      queueNo,
      task: () => {
        asyncFunction().then(value => resolve(value)).catch(err => reject(err));
      }
    });
  });
  if(!doNotStart && !this.job) {
    await this.start(true);
  }
  return myPromise;
}
```

Same parameters as before. We accept an async function - this is a thunk delaying the operation of the actual function (so `fetch('https://www.google.com')` becomes `() => fetch('https://www.google.com')`). We increment and get a ticket number which we call `queueNo`, then push the job onto our local list with the queue number. If the queue isn't running, and `doNotStart` isn't true, we start the queue. The job is pushed and we return a promise that resolves when the job compeletes.

```javascript
decorator(asyncFunction, doNotStart) {
  let thisQueue = this;
  return function () {
    return thisQueue.add(() => asyncFunction.apply(this, arguments), doNotStart);
  }
}
```

Same as before, the decorator accepts an async function, then returns a new async function that does all the thunking and enqueueing for you, so you can use it instead.

## Pause and unpause

Pause and unpause are quite simple, since the complexity is pushed onto the actual queue execution function:

```javascript
async pause() {
  await this.setAsync('paused', true);
  this.job = null;
}

async unpause() {
  await this.setAsync('paused', false);
  if(!this.job && this.index < this.queue.length)
    this.start(true);
}
```

Pause is just setting the shared paused variable to true, and unpause is doing the opposite and starting the queue again.

## Running

Finally, the meat and potatoes. The core of the queue's functionality is in the `runNext` function, which schedules the next job and executes the current one. The `start` function from before simply runs `runNext` with the correct timeout, and the rest happens here. It's a large unwieldy function, so let's go over it in pieces:

As before, runNext returns a function that is then passed to setTimeout. This is so the values are frozen inside the closure, but this function needs to be a non-async function. setTimeout has weird issues when running async functions, so we consolidate our asynchronous operations within this function and use `.then`. Makes for more unreadable code, but it executes the same way. When illustrating the code, I'll use await and omit the error handling and value parsing compatibility code, but you can find the final function [here](https://github.com/hrishioa/sharded-interval-queue/blob/master/sharded-interval-queue.js). I'm also localizing `this` inside the function for the closure, but I'll omit that here for readability. `this` will refer to the queue we're in.

First, we check to see that the queue is meant to be running, and increment the index:

```javascript
let paused = await this.getAsync('paused');

if(paused || this.index >= this.queue.length) {
  this.job = null;
  return;
}

this.index++;
```

Next, we check if the current job still has a slot in the global queue to guard for pause conditions. If the queue has already passed the current job, we get another ticket and requeue it, and call runNext again:

```javascript
if(this.queue[this.index-1].queueNo < current) {
  this.increment('ticket').then(queueNo => {
    queueNo = parseInt(queueNo);
    this.queue.push({
      queueNo,
      task: this.queue[this.index-1].task
    });
    this.runNext()();
  });
}
```

If not, we set a timeout for the next instance of `runNext`, and run the current job:

```javascript
if(this.index < this.queue.length) {
  this.job = setTimeout(
    this.runNext(),
    (this.queue[this.index].queueNo-current-1)*interval);
} else {
  this.job = null;
}
this.setAsync('current', this.queue[this.index-1].queueNo).then(() => {
  thisQueue.queue[this.index-1].task();
});
```

We calculate the interval to the next queue number based on the current one. We don't have the problem of queue numbers going stale anymore since we no longer derive them from time, and we get the time-based distance between jobs. For a queue of interval 1 second, jobs 1 and 5 have 4 seconds between them, jobs 2 and 3 have one second between them, and so on.

Once this is done, we set `current` to the job number that is currently executing, and then run the job. This is the most problematic operation for parallel state modification, but the max distance that the variable can move is the time it takes for the setTimeout assignment to execute divided by the queue interval, and in my runs I haven't had a problem. This is partly because the lowest resolution we can get for queue execution is 1 millisecond, and any corruption here will be quickly fixed by the next `runNext` call. If we wanted more parallel safety or were moving below 1 ms, we could atomically increment `current`, then check to see if the value was valid and only do the assignment if it wasn't. Or, if the adapter supports it, we could set this value atomically.

This is the entirety of `runNext`. Next, we move on to storage adapters.

## Storage Adapters

Like I mentioned before, replacing the global object with a custom adapter simply involves replacing the five functions in the queue. I've provided [LowDB](https://github.com/typicode/lowdb) and [Redis](https://github.com/NodeRedis/node_redis) implementations and a way to add custom adapters.

For lowdb, we use the `initStorage` function to make sure we have a valid JSON file to write to. The other functions are implemented as shown below:

```javascript
this.initStorage = async () => await db.defaults(dbDefaults);
this.getAsync = async key => await db.get(this.queueName).get(key).value();
this.isSet = async key => await db.get(this.queueName).has(key).value();
this.setAsync = async (key, value) => (await db.set(`${this.queueName}.${key}`, value).write())[this.queueName][key];
this.increment = async (key) => (await db.get(this.queueName).update(key, n=>n+1).write())[key]-1;
```

For Redis, most of the operations are the same, except for two. We don't need an `initStorage` function, and we use a lua script to increment a variable atomically:

```javascript
this.initStorage = async () => null;
this.getAsync = async key => await redisGet(`${this.queueName}_${key}`);
this.setAsync = async (key, value) => await redisSet(`${this.queueName}_${key}`, value);
this.isSet = async key => (await redisGet(`${this.queueName}_${key}`)) !== null;
this.increment = async (key) => {
  const lua = `
            local p = redis.call('incr', KEYS[1])
            return p`;
  return await redisEval(lua, 1, `${this.queueName}_${key}`);
}
```

For a custom adapter, the `setStorageCustom` function accepts custom implementations for the queue. A small change here is that the queue's functions do not accept a queue name, but I'd later wanted the ability for custom implementations of connecting queues to keys. Custom adapter functions accept a queueName parameter, which is then wrapped to be compatible with the internals of the queue.

```javascript
this.isSet = async (key) => isSet(this.queueName, key);
```

## Testing

And that's it for the implementation. A testing function is implemented in [test.js](https://github.com/hrishioa/sharded-interval-queue/blob/master/tests/test.js), as well as tests for [redis](https://github.com/hrishioa/sharded-interval-queue/blob/master/tests/test_redis.js) and [lowdb](https://github.com/hrishioa/sharded-interval-queue/blob/master/tests/test_lowdb.js), where you can vary the number of active queues, number of jobs and the number of pauses to see how it affects performance. Redis proved to be the fastest implementation with a shared state, which was no surprise.

This has been a fun project, and to my abject surprise has gotten a lot of of use in the week it's been up. I'm glad to see it being used in other projects. It also makes me worry about the correctness of my implementation, and I'm hoping this leads to feedback I can use to improve it. A lot of things are good enough for your own use, but I'm always nervous when someone else tells me they're using my package in production.