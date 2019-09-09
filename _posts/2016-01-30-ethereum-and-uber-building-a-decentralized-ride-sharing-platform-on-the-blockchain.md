---
layout: post
title: "Ride-sharing on the Blockchain"
date: 2016-01-30 20:39:37
image: 'Ethereum/banner.jpeg'
share_image: 'Ethereum/banner.jpeg'
description: "Exploring the Concept of ride-sharing on the Ethereum Blockchain" 
tags: 
- Cryptocurrency
- Ethereum
- Blockchain
- Uber
- HacknRoll
categories:
- Hackathon Stories
twitter_text:
---

I've recently become quite interested in [Ethereum][ethlink], the decentralised software platform that uses the underlying concept of the Bitcoin blockchain to run code while maintaining trust. I've been playing around with Smart Contracts, which are modules executed by the blockchain. I'm not going to describe in detail how the system works, but there is a [starter guide][ethredditstarter] on the Ethereum subreddit that you can look at for more information.

During [HacknRoll 2016][hacknroll], the idea we came up with was to make use of Ethereum to implement a ride sharing service, much like Uber. Originally, the idea was to implement service-sharing, but we felt that Uber was a prime test case for decentralization. The company has been steadily [cutting][ubernews] its fee for drivers, and the incident during [hurricane Sandy][ubernews2] served to demonstrate the perils of centralisation in a peer-to-peer system.

Here's the first part, where I will attempt to illustrate the theory behind the system.

## The Concept

We wanted to build a system that was as close to a closed system as possible. The central management system would only take as much as it needed to operate, and the rest would be distributed among users. This would achieve the lowest prices that the market can bear, with a middleman not beholden to corporate motivations. The rider would pay the driver, and that was it. We immediately sketched out the basics, and you can see the overall structure below.

![Whiteboarding]({{site.url}}/assets/img/Ethereum/whiteboard.jpg)

The most important segment is of the two contracts that exist on the blockchain. The first, called **the Central Exchange**, is the central system that decides the current market price per mile, and keep track of the stock (I'll explain later) and other metadata related to each consumer account. The second, called **the Rideshare**, connects to the exchange when needed in order to retrieve prices. This is factored into the calculation of reputation points. There are two primary systems at work here, which will be explained below.

### Stock

The first problem we attempted to tackle is that of immutability. Any algorithm residing on the blockchain cannot be changed externally. It must decide to destroy itself (via an operation called a `suicide`) after telling any contracts that need it to call someone else. This needs to be done by collective consensus, and for this purpose we came up with a stakeholder system whereby each user - either driver or rider - is given points of stock that correspond to how many kilometers they've ridden using the system. Stocks are issued using hashes, and are re-weighted when more stocks are issued. These stocks will decide the amount given back when the system receives a profit above a fixed amount. They will also determine your voting power in making a decision about changing the central system. The voting system is fairly complex and wasn't implemented for the purpose of the hackathon, but we are considering implementing it in the future.

In a full-fledged implementation, there will be a third algorithm on the blockchain that opens and closes possible issues for voting. Each user is given a small amount as incentive for voting, and a multiple of that when the issue is resolved if he or she is on the winning side. This incentivizes voting and disincentivizes random voting in order to collect fees.

A rewrite of the system - as we foresee it - would involve one or more users posting rewritten source code online, which will be reviewed and voted on. When accepted, the system will then draw the code and bytecode, instantiate a new version and destroy itself while updating client algorithms to point to the new version.

### Money and Reputation Points

The rideshare system works by allocating reputation points and in-system currency to the user. For now we have made use of a custom currency, but it seems possible to implement trusted payment gateways to remove the barrier of cryptocurrency should the user so wish. Both are stored in the rideshare system, and the blockchain ensures trust and takes care of double-spending issues.

## Example

An example ride would be comprised of the following steps:

1. A driver joins the system using a secure authentication hash (implemented by a trusted third party) of his Driver's license.
2. The driver posts a contract on the rideshare system, specifying his current location, price per mile and limit (in miles) of how far he is willing to go. It holds his reputation points in escrow and posts the contract.
3. A client-side interface (can be implemented by anyone) pulls this informaton by analyzing the transactions to the blockchain.
4. A user selects from available contracts in his region, and relays this information to the rideshare system.
5. The system checks to see if the contract is still valid (uncancelled) and that the accounts are valid, i.e. they exist and that the user has enough money in the account to cover calculated costs and transaction fees.
6. The contract is successfully accepted by the system, at which point the user's reputation points and the total cost of the ride are held in escrow.
7. The driver and the user successfully complete the ride.
8. The system receives confirmation from both the driver and the user that the ride has been completed, at which point the reputation points are released, recomputed as per the current market price (there is a dynamic penalty for exceeding the current rate) and the rating from each other, and the driver is paid.
9. A request is sent to the central exchange to issue stock to the driver and rider, along with the remaining transaction fees that weren't spent during computation.
10. The central exchange stores the fees received in an account, and checks to see if it exceeds a dynamically calculated maximum based on the current gas (the metric for computation on the Ethereum blockchain) price.

Both the driver and the user are obligated to complete the ride, as the reputation points are required to register or accept a contract on the system. They can cancel the ride at any point and receive some of these points back, but the computation will assume that they have been given a rating of zero by the other party. 

In such a system, the only money being used is the amount spent on transactions. The nodes executing the contract are paid for their time and energy, but the rest of the capital in this economy resides with either the driver or the rider. 


[ethlink]:https://www.ethereum.org/
[ethredditstarter]:https://www.reddit.com/r/ethereum/comments/3vxvlx/starter_guide_almost_all_the_links_youll_need_to/
[hacknroll]:http://hacknroll.nushackers.org/
[ubernews]:http://therideshareguy.com/should-you-still-drive-for-uber-after-the-latest-rate-cuts/
[ubernews2]:http://fortune.com/2012/11/02/uber-nyc-and-the-sandy-surge/