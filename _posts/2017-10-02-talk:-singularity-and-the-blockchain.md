---
layout: post
title: "Talk: Singularity and The Blockchain"
date: 2017-09-20 05:37:05
image: 'FAYATalk/watchmen.jpeg'
description: A shallow dive into blockchain technology and what it means for the singularity of tomorrow
tags:
- Singularity
- Blockchain
- Bitcoin
- Ethereum
- Talk
- FAYA
- Singularity University
categories:
twitter_text:
---

This is a talk I had been meaning to do for a long time. Having collected enough material over collected rants with friends (seriously - there have been hundreds of pages of scribbled notepads and conversation material still taking up space on my desk), when the good folks at FAYA and [Singularity University](https://su.org/) asked for some time on Blockchain, I thought it might be a good chance to explore some of these ideas further.

{% include youtubePlayer.html id="muQFOuEiAFo?t=42m43s" %}

I had originally hoped to leave the video here as a point of discussion, but the audio is spotty and doesn't seem to be an enjoyable listening experience. For the undaunted, below is a basic sum-up of the ideas discussed:

## The Singularity

Now the singularity is one of the most polarising concepts I've encountered in my time - both in knowing what it is, and whether you think it's hogwash. Personally I'm still in two minds as to which side I'm personally on - I see arguments for both sides, and I support hedging against both, but I also see the value in considering it deeper.

For a quick recap, the Singularity in physics is the point where existing models break down - space becomes so tiny and so dense that both existing models for the physical world, quantum mechanics and general relativity, come into conflict. At this point, all bets are off as to what happens next. 

The Technological Singularity is a term first used by Von Neumann circa 1950 and driven to popularity by Ray Kurzweil and his treatise [The Singularity is Near: When Humans Transcend Biology](https://en.wikipedia.org/wiki/The_Singularity_Is_Near). It refers to something similar to the Singularity in the Physical Sciences: it is the point when all calculations of the future fail to compute. Driven by ever-accelerating technological progress, proponents of the Singularity theorise that we are on our way to an event - many propose this will be the conception of an [Artificial General Intelligence](https://en.wikipedia.org/wiki/Artificial_general_intelligence) - beyond which the future of the human race is unclear.

Much like Dr Manhattan and his inability to see past the very event he means to stop, this leads to a few factions emerging: The first are the strong supporters of the theory, whose ranks include Stephen Hawking and Elon Musk. They quite loudly ask the human race to slow down and implement checks that can prevent such an occurrence. Meanwhile, the opponents claim this will only imped the progress of human technology. Finally, I along with the rest of us wish to study it further before we make our judgement if one is to be made after all.

Let's save our judgement for later: Below is a quick look at the technology that underpins the blockchain, and how it might be relevant to said question.

## History

The first part of the talk is a brief history of blockchain, and how it is older than some of us think. Blockchains are essentially book-keeping systems, and following their history leads us from [single-entry bookkeeping](https://en.wikipedia.org/wiki/Single-entry_bookkeeping_system), [double-entry bookkeeping](https://en.wikipedia.org/wiki/Double-entry_bookkeeping_system) to [triple-entry bookkeeping](https://en.wikipedia.org/wiki/Momentum_accounting_and_triple-entry_bookkeeping). As we follow this progression, the systems employed start to look suspiciously like the trustless ledgers we have all come to accept and love. Strange enough for a system whose champion application is an [electronic payment system](https://en.wikipedia.org/wiki/Bitcoin) that eschews anything from the old financial world...

## Breakthroughs

![sybil]({{site.url}}/assets/img/FAYATalk/sybil.jpg)

The biggest breakthrough (in my opinion) that blockchains pioneered was a solution to a problem any trustless peer-to-peer system faced: [The Sybil Attack](https://en.wikipedia.org/wiki/Sybil_attack). Named around 1973 in a book about a woman who had multiple personalities, the sybil attack is the problem of countering adversaries that might portray themselves as multiple people in a trustless network. In essence, solving the problem is finding a counter to ensuring that any user cannot represent himself as multiple entities that are valued the same to the network. Splitting yourself in half MUST mean a split in value of about the same, or the system fails. 

Blockchain's solution has been to use the Proof-of-Work system, wherein participants burn energy by solving cryptographic puzzles in order to maintain the network. Unfortunately, this currently has the entire bitcoin network *burning* energy at about the same rate as Ireland, with no end in sight.

## Singularity and The Blockchain

Moving on, how are blockchains useful to a world approaching the singularity, or even post-singularity?

Effectively, blockchains enable a way for trustless systems to agree on eventual truth. In the case of the majority of blockchains, this is the universal idea of time - what came before, and what will come after. The block and chain system is essentially a way of establishing the order of things as they happen in the physical world. A transaction in a later block essentially indicates it being after a certain time, in a stochastic sense. Most systems will find, that once a single variable is agreed upon - like time - that others can be derived from it.

Why is this important? After all, you're perfectly comfortable checking your clock on the wall, your phone, or your watch for the time - or looking it up online. You are also comfortable asking a bank to hold your money, even those of us that know that few if no banks in the world keep all of their deposits on hand as assets. The human world works on trust, and on centralized systems. Despite their many shortcomings, they are easy, they rely on human nature and self-correct over time, as we can see from the history of the world. 

However, machines are unblessed with this state of certainty. Decoupled from the outside world and the intuitive sense of forward and backward given to us by evolution, machines are often in need of specific and attack-proof rules by which to determine their reality. For example, we've measured a single second as 9,192,631,770 cycles of radiation emitted from a Caesium-133 atom. For all the rules of physics that we know of, this holds true at any point on the earth and in the universe (discounting or accounting for relativistic effects).  Giving this measurement to a machine, like fire burning in a fennel-stalk, enables it to measure the seconds that go by knowing that any other with the same measuring stick with measure the same.

However, two problems arise. One - how does the system know what came before and what came after?  On the one hand, it can create - relative to its existence - some idea of past and future. But on the other hand, it cannot create one from an objective standpoint. It cannot know that the distance between each second was merely a second, or that it had been turned off or locked in a process between a significantly longer second. Two - it cannot then verify the claims of other machines, or parts of itself, of their ideas of past and present. This is already becoming a complicated issue, and we haven't discussed the possibility of malignant actors in the system who will do anything they can to alter the course of it.

And so we arrive at the problem, and why decentralized database technology is important. Moving towards a machine-run world, we are quickly outpacing the human standards of old. There needs to be a way that multiple actors - machines or humans - can agree on a common truth, lacking any common factors. Blockchains pave the way for such a future - albeit in an imperfect fashion - by solving some of these problems. I'll let the talk complete that part.