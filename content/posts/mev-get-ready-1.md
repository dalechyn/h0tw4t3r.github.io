---
title: "🕵️‍♂️ Get Ready | MEV Series 1/n"
authors: ['h0tw4t3r.eth']
series: ['mev-searcher']
tags: ['mev', 'ethereum']
slug: "mev-get-ready-1"
featuredImage: "images/dark-forest.jpeg"
date: 2022-08-21T20:32:36+03:00
ID: 7a27be1d5746668a2101945fc045f50f
---

### 👋 Greetings Reader!
In this post, I will cover my personal MEV setup experience and what I needed for successfull MEV Extraction.

Before jumping right in the forest, you need to make sure your backpack is filled with minimum knowledge of MEV, and that you have your [MEV Backpack](/posts/mev-backpack) on!

## 🤔 My first MEV encounter
My first touch to MEV was the discover of [**Ethermine's arbitraging bot**](https://etherscan.io/address/0x5aa3393e361c2eb342408559309b3e873cd876d6).

He backruns (put's his transaction right after one is landed) public DEX trades and equalizes pools in which the trade occured. Kind of like an on-chain sanitizer so to say.

!["Ethermine's bot"](/images/mev-1/ethermine.png)

Not bad, not bad. Actually, at the time I discovered this bot, he had around **15M$** net-worth.

I also did not know at the time, that this bot is ***Ethermine's***, lol.

## 🕵️‍ The Research
I started from reading the transaction data and tried to understand what bot passes.

Let's check [**this**](https://etherscan.io/tx/0x8e060ca91df454f2bb24f8348853a97859ae580234a98ce90c16c158825d77fb) transaction as an example:

!["Arbitrage"](/images/mev-1/arb.png)

Here we clearly see, that bot swaps `2.463801181402802852 ETH` through two pools getting `2.481955914265819052 ETH` at the end, with the profit of ~ `0.01815473286 ETH` (not taking the fee costs in account).

So how does this work?

Let's see the tx data (for better view the data is split by slots – 32 bytes (64 hex symbols)):
```
0x00000052 - Method Signature
0000000000000000000000000000000000000000000000000000000000eac4bb - Block Number
00000000000000000000000000000000000000000000000022312e1ee6b3aaa4 - 2463801181402802700 WEI that bot estimates to arbitrage 
000000000000000000000000a0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 - USDC address (a.k.a. the quote token)
00000000000000000000000020e95253e54490d8d30ea41574b24f741ee70201 - First Pool Address (Uniswap V2 Pair-like)
0000000000000000000000008ad599c3a0ff1de082011efddc58f1908eb6e6d8 - Second Pool Adddress (Uniswap V3 Pool)
0000000000000000000000000000000000000000000000000000000000000001 - Type of the first pool (0=UniswapV3, 1=UniswapV2)
0000000000000000000000000000000000000000000000000000000000000000 - Type of the second pool (0=UniswapV3, 1=UniswapV2)
```

Wasn't that hard you may say? I wish it wasn't back then.
> Bro rly?

We can clearly see that bot does not pass WETH9 address, which means that he trades to only get profits in ETH.
Also we can see that the actual swapped amount is a bit different, which means that the total input amount is being recalculated when the transaction is executing.

## 📝 Overall
This bot attracted my interest, as it does approximately 100-300 txs per day, earning >10,000$ per day.

Yet then, I decided to explore MEV opportunities.

In future post I'll cover my plan at the time, with some pseudo-code alpha.

If you want to see more content like this, subscribe to my twitter [0xh0tw4t3r](https://twitter.com/0xh0tw4t3r).
