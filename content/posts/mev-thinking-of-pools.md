---
title: "MEV | 🏊 Thinking of Pools"
authors: ['h0tw4t3r.eth']
series: ['mev-searcher']
tags: ['mev', 'ethereum']
slug: "mev-thinking-of-pools"
featuredImage: "images/mev-2/pool.jpg"
date: 2022-08-26T20:00:57+03:00
ID: 0ef000549a64a48921145c2cd2a72b1a
katex: true
---

## 👋 Greetings Reader!
In this post, I will cover my personal MEV Extraction planning steps.

Before reading, it's better to check [previous post](/posts/mev-get-ready) in this series if you haven't.

---------------------------
## 📃 The Plan...

...did not exist at the time at all.

Considering the very narrowed amount of sources regarding how to **actually** come up with a list of actions, if any, I came up with next problems:


- [x] **Research UniswapV2 contracts**

    It's a good starting point as it is one of the easiest AMMs, and yet it has many [vampire](#-vampire-attack)-clones
> Checked as I had experience with integrating it into various Smart Contracts.

- [ ] **Research UniswapV3 contracts**

    Yep, Uniswap agin. It's not a secret that the Uniswap is the most popular DEX so far.

- [ ] **How to find arbitrage opportunities/unbalanced pools?**

    This was one the toughest.

    Initial idea was to ~~host a subgraph on local node~~ spin up a custom subgraph on [**TheGraph**](https://thegraph.com/);

    I thought that with a slightly modified [Uniswap V2 Subgraph](https://github.com/Uniswap/v2-subgraph) I can collect the Top 1000 pools by 15-minute liquidity, find neighbours from [Uniswap V3 Subgraph](https://github.com/Uniswap/v3-subgraph) and pass them to my bot to check wether there are arbitrage opportunities.
    > Silly me thought this could really work...

    My modified subgraphs are available [here for V2](https://thegraph.com/hosted-service/subgraph/mevspace/mevspace-uniswapv2), and [here for V3](https://thegraph.com/hosted-service/subgraph/mevspace/mevspace-uniswapv3).

- [ ] **Find a way to pass the transaction so it's not stolen by other bots**

    That was an easy one – [**Flashbots**](https://docs.flashbots.net/) came for help! If you haven't heard about them, I wrote a really short explantaion [here](#-flashbots).


---------------------------
## 🧛‍♂️ Vampire Uniswap V2 specifics

So how the "vampires" are different from it's dracula?

Generally, the diff lies in the fees, which are **hardcoded**.

Let's take a look at this part of code in [`UniswapLibrary.sol`](https://github.com/Uniswap/v2-periphery/blob/0335e8f7e1bd1e8d8329fd300aea2ef2f36dd19f/contracts/libraries/UniswapV2Library.sol#L46-L48), the place where fees are defined:
```solidity
uint amountInWithFee = amountIn.mul(997);
uint numerator = amountInWithFee.mul(reserveOut);
uint denominator = reserveIn.mul(1000).add(amountInWithFee);
```

`997` – is the fee numerator, and `1000` – is the fee denominator. That's where we see the $$\text{fee(UniswapV2)} = 1 - \frac{997}{1000} = 0.03$$.

And now, let's take a look at [`PancakeLibrary.sol`](https://github.com/pancakeswap/pancake-smart-contracts/blob/d8f55093a43a7e8913f7730cfff3589a46f5c014/projects/exchange-protocol/contracts/libraries/PancakeLibrary.sol#L70-L72):
```solidity
uint256 amountInWithFee = amountIn.mul(9975);
uint256 numerator = amountInWithFee.mul(reserveOut);
uint256 denominator = reserveIn.mul(10000).add(amountInWithFee);
```
As you probably catch, the fee here is the next:
$$\text{fee(PancakeSwap)} = 1 - \frac{9975}{10000} = 0.025$$


**The problem** is that the fees are **hardcoded**, and there's no straight way to pull them out.


I use TypeScript, so I went with Uniswap's [V2](https://github.com/uniswap/v2-sdk) and [V3](https://github.com/uniswap/v2-sdk) SDKs and I had to do a couple of patches to get this working with different fees. Write in the comments if I shoud post them, or hit my dm on [Twitter](https://twitter.com/0xh0tw4t3r)!

---------------------------
## 📃 Finalized plan:
- [ ] **Start a Typescript Project**
- [ ] **Make specific patches to Uniswap SDK**
- [ ] **Come up with V2-V2 fee-driven balancing solution**
- [ ] **Come up with V2-V3 fee-driven balancing solution**
- [ ] **Listen for top pools and look for arbitrage opportunities**
- [ ] **Prepare the bundle to be sent and rekt the pools**
---------------------------
## 📝 Overall
In the next post I'll uncover UniswapV2 balancing solution, and why my plan was full of crap.

If you want to see more content like this, subscribe to my twitter [0xh0tw4t3r](https://twitter.com/0xh0tw4t3r).

---------------------------
## 📚 Glossary

#### 🧛‍♂️ Vampire Attack
> To put it simply a vampire attack is when one DeFi protocol offers better rates to attract investors from another platform. Now one of the most famous vampire attacks happened with SushiSwap, they were simply able to offer one of the best liquidity provider rates to any investor on their platform. Doing so meant a lot of people pulled their liquidity or their money from Uniswap and then put it in SushiSwap instead.
> 
>– https://omerkeman.medium.com/what-is-a-vampire-attack-in-crypto-fdfc5e1fc5fc

#### 🤖 Flashbots

>Flashbots is a research and development organization working on mitigating the negative externalities of current Maximal Extractable Value (from now on MEV) extraction techniques and avoiding the existential risks MEV could cause to state-rich blockchains like Ethereum.
>
>– https://docs.flashbots.net/

From software POV: Adds new methods to GETH API – `eth_callBundle`, `eth_sendBundle`, `eth_sendMegabundle` and `eth_sendPrivateRawTransaction`.

As a MEV Searcher, you will be interested in `eth_callBundle` and `eth_sendBundle`

`eth_callBundle` – allows you to simulate the bundle, which may consist of multiple transactions, against a certain block which was mined.

`eth_sendBundle` – allows you to send your bundle to MEV Relay, where you can specify the maximum block where one can get included.
