---
layout: post
title: "The two innovations of Hashmasks"
date: 2021-02-13 21:12:27
image: Hashmasks/cover2.png
description: "Understanding new forms of digital art and the possibilities"
tags:
- Blockchain
- Ethereum
- NFTs
- Hashmasks
categories:
- Emerging Fields
twitter_text:
---

It's not a secret to the people who know me that I have been quite excited about [The Hashmask Project](https://www.thehashmasks.com/). Still relatively unknown in the non-NFT world, Hashmasks have single-handledly revitalized the space and built value - with a single Hashmask now going for *at least* 1650 USD per mask at the time of writing.

However, from the outside looking in - and even to most insiders - it can be hard to see why the project is of landmark significance among NFTs, and has two key innovations that will be important to the space in the years to come. Here's what they are.

(As a side note, if you're interested in checking out some of these for yourself, you can find them on [OpenSea](https://opensea.io/collection/hashmasks) or on a community-built tool to explore masks, called [Maskinson](https://maskinson.netlify.app/?0x42d97b2d395c6ab73c43c37e9d9789261dce0b8f).)

## What are NFTs?

Before we begin, a small primer on what all of this means. NFTs - or Non-fungible Tokens - are a way of representing assets (primarily on blockchains) in a way that indicates their non-fungibility. Most of the crypto assets you've heard of (BTC, ETH, Doge) are fungible tokens - 1 bitcoin is equal to another bitcoin, and they're equal to two others, no matter where you found them or who holds them - much like currencies. This fungibility is an important property of currencies that is often the primary reason for their existence. Trading two cows is hard when you have no base fungible asset to value them in, leading to an illiquid market. Denominate them in USD - where one is worth 200 USD and the other is 350 - and you suddenly have the ability to swap them in an equitable fashion, as well as establishing their value against assets of other types.

Non fungible tokens are fundamentally not equal from unit to unit. They might be minted in the same place, but they usually hold something (within their code or data) that makes them unique. None are equal to another. It stands to reason that a good use for such tokens can be found in art, where each piece of art is separate and different in worth (both personal and global) to another. As an example, all Pokemon Cards have similar properties - two sides, four edges, made of paper - but are not equal in value due to their contents. It's the exact shape and number of tails of the animal printed on the card that determines value.

![Pokemon Cards]({{site.url}}/assets/img/Hashmasks/pokemons.jpg)

A number of standards have evolved - most notably in the Ethereum blockchain - to denote these tokens. The most prominent one currently in use is [ERC-721](https://eips.ethereum.org/EIPS/eip-721), but ERC1155 and many others are in development and use. In essence, these standards denote the most common operations such tokens need to have, and an interoperable interface for using them in standardised marketplaces. For a Non-fungible token to exist, it needs to:
1. Be mintable, i.e. it needs a method of creation that can ideally be regulated
2. Be transferrable - included here is also the concept of ownership where each token has a designated owner
3. Be inspectable to it's uniqueness - a way to understand what makes the token unique, and how it can be valued

Some optional properties may include a way for the token to be burnt, where it is provably removed from existence, or the ability to modify the asset after creation.

[Rarible](https://rarible.com/) or [OpenSea](https://opensea.io/), two of the most prominent marketplaces for NFTs, can provide some live examples of such tokens.

![Art on Rarible]({{site.url}}/assets/img/Hashmasks/rarible.gif)

## What are hashmasks?

Hashmasks are 16,384 unique (actually 16,375 due to a [bug in the creation process](https://thehashmasks.medium.com/the-curious-tale-of-mansa-and-jim-faab29181b2)) artworks created by over 70 artists, where multiple traits (mask, skin color, eye color, background, etc) are combined to create the final piece. Here are a few examples, the first of which is a Steel skinned, glass eyed, robot abstract mask with a shadow monkey:

![Some Masks]({{site.url}}/assets/img/Hashmasks/SomeMasks.png)

You can [read more about the history of their creation here](https://thehashmasks.medium.com/masterpiece-decentralized-artistry-57a186d4e1c9), but the masks were generated through a combination of automated and manual processes, combining the work of many artists. Each mask is an ERC-721 token, holding information about the mask, its owner, traits and a link to the image that the token represents. The images are stored on the IPFS decentralized file storage system - [here](https://ipfs.io/ipfs/Qmbz5QGvvFQw2hi3KM6us3Vds5QBT7bgX8pZFXPWoNbiYh) is the higher resolution, original version of the mask above.

Despite being interesting pieces of art themselves, hashmasks bring us one step closer in the world of NFTs to answering the question: what is the point of an NFT? Few fatally-flawed exceptions aside, NFTs have been used to describe and encapsulate purely digital assets, whose contents have mostly been revealed in their entirety on a public blockchain. If the artwork has value, what stops an individual from copying it and having it for themselves? Case in point: [you can download the mask above]((https://ipfs.io/ipfs/Qmbz5QGvvFQw2hi3KM6us3Vds5QBT7bgX8pZFXPWoNbiYh)) from IPFS, print it, use it as a wallpaper, print it, frame it and hang it on your wall. This is not the case for a physical painting, where a copy is simply not the same due to the limitations of our physical world. For a digital entity, a copy is the same as the original.

Or is it? NFTs have a long way to go in establishing themselves as a medium, and in establishing ways in which the original is superior to a copy. This is a question whose answers have wide-ranging implications for broader digital media and piracy. Let's have a look at what Hashmasks do that separate them from most NFTs out there. The first, and most important, is:

## Connecting a non-fungible asset to a fungible token

There are only two configurable properties about a Hashmask: ownership - who owns it - and name. Each Hashmask can be assigned a unique name, but the only way this can be done is by burning [NCTs](https://www.coingecko.com/en/coins/name-changing-token#markets) or [Name Change Tokens](https://etherscan.io/address/0x8a9c4dfe8b9d8962b31e4e16f8321c44d48e246e) - fungible [ERC20](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) tokens on the Ethereum blockchain - 1830 NCTs to be exact. Each mask will emit (or mint) 10 NCTs per day until 26 January 2031, and these tokens are truly fungible once claimed, and can be traded freely.

I believe this is a key innovation - perhaps borrowing from other digital media like video games, where in-game assets can be created and traded (EVE Online's Frigates and Cruisers come to mind) - which allows a non-fungible asset to be connected to a fungible one. The benefits are interesting.

![NCT on Uniswap]({{site.url}}/assets/img/Hashmasks/UniswapNCT.png)

First, this provides an advantage to an original that a copy does not have. Second, connection to a fungible asset with a liquid market (the [NCT contract on Uniswap](https://info.uniswap.org/pair/0x9f4aa9b4661f0c55b61fc12b1944f006a71c773f) holds liquidity worth $294,364 USD and trades over 1.2M USD in volume per day, at the time of writing) sets a price floor and provides a starting point to establish value for the underlying asset. The tokens also open up possibilities for new art to be modifiable and interactive, which brings me to my next point. Hashmasks, as an NFT, succeed in:


## Taking advantage of the medium

![Girl With Balloon]({{site.url}}/assets/img/Hashmasks/shred-1.gif)

This is of course, not an original concept - multiple projects like [CryptoKitties](https://www.cryptokitties.co/) and [Gods Unchained](https://godsunchained.com/) have done this before. However, it's still not the norm, and I believe Hashmasks will be the tipping point - or I certainly hope so.

The key differentiator in an NFT is that it is living code - it exists within the confines of a turing-complete execution engine, and can define unbelievably complex methods that mutate its existence and contents. While a piece like [The Girl With Balloon](http://www.sothebys.com/en/auctions/ecatalogue/2018/contemporary-art-evening-auction-l18024/lot.67.html?locale=en) needs to employ complex manual methods to mutate itself, an NFT can encompass transparent code that can respond to any number of stimuli, as long as it remains on-chain.

The best and most successful artwork of our time are those that take true advantage of the medium of expression, and what sets the medium apart from all others. Oil paintings can play with light in ways no other medium can, video games can provide first-person immersion unlike any movie, and a movie on the big screen can tell stories with artistic direction that VR cinema will struggle to reach for a long time. An examination of [Why the masterchief works](https://www.youtube.com/watch?v=seefNNPNGmM) and [The Kuleshov Effect](https://www.youtube.com/watch?v=Vy2Vhnqtu8I) come to mind as good examples here.

Sadly, the NFT space is still mostly pictures and GIFs trying to sell you that the concept of ownership - tied to an large hex number on a blockchain - is enough to be worth millions of dollars. And the space for interactive NFTs that take advantage of unlimited interaction with the user and the art itself, is still open. Almost as open as the space for expression within a turing machine.

I think we are at the precipice of stories that write themselves through their history of ownership, movies that emit their own collectibles, and many more things I'm limited by my imagination to see. I'm confident that the artists, who make this space what it is, will not be.

