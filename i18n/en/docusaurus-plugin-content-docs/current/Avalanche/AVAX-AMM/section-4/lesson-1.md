---
title: Completion and Next Steps
---

## Congratulations! 🎉

You have successfully built a fully functional AMM (Automated Market Maker) on Avalanche! Let's recap everything you've accomplished in this project.

## What You Built

### Smart Contracts

- **ERC20 Token Contracts** — Two custom ERC20 tokens (USDC and JOE) to use as trading pairs in the pool
- **AMM Contract** — A complete AMM implementation featuring:
  - Liquidity provision (`provide`)
  - Liquidity withdrawal (`withdraw`)
  - Token swapping (`swapTokenX`, `swapTokenY`)
  - Price estimation functions
  - Share-based liquidity tracking
  - Slippage protection

### Frontend Application

- **Wallet Connection** — MetaMask integration using ethers.js
- **Provide Liquidity UI** — Interface for depositing tokens into the pool
- **Withdraw Liquidity UI** — Interface for removing liquidity and receiving tokens back
- **Swap UI** — Interface for swapping between tokens with real-time price estimates
- **Pool Details** — Display of current pool state including reserves and share information

### Deployment

- Deployed contracts to the **Avalanche Fuji C-Chain** testnet
- Configured the frontend to interact with deployed contracts

## Key Concepts Learned

### The Constant Product Formula

```
x * y = k
```

This simple equation is the foundation of AMMs like Uniswap. It automatically adjusts token prices based on supply and demand without needing a centralized order book.

### Liquidity Pools

Liquidity providers deposit pairs of tokens and earn a proportional share of the pool. In production AMMs like Uniswap, providers also earn trading fees (typically 0.3% per swap).

### Impermanent Loss

As a liquidity provider, you should be aware of **impermanent loss** — the temporary loss in value compared to simply holding the tokens. This occurs when the price ratio between the two tokens changes significantly after you provided liquidity.

### Slippage

The price impact of a trade depends on the size of the trade relative to the pool size. Large trades cause more slippage. Our implementation includes slippage protection via the `minAmount` parameter.

## How to Extend This Project

Here are some ideas for extending what you built:

### 1. Add Trading Fees

Modify the swap function to take a small fee (e.g., 0.3%) and distribute it to liquidity providers:

```solidity
uint256 constant FEE_NUMERATOR = 997;
uint256 constant FEE_DENOMINATOR = 1000;

// Apply fee to input amount
uint256 amountXWithFee = amountX * FEE_NUMERATOR;
uint256 amountY = (totalAmount[_tokenY] * amountXWithFee) /
    (totalAmount[_tokenX] * FEE_DENOMINATOR + amountXWithFee);
```

### 2. Support Multiple Trading Pairs

Deploy multiple AMM contracts, each with a different token pair, and build a router contract that can route swaps across multiple pools.

### 3. Add Price Oracle

Implement a time-weighted average price (TWAP) oracle using cumulative price tracking:

```solidity
uint256 public price0CumulativeLast;
uint256 public price1CumulativeLast;
uint32 public blockTimestampLast;
```

### 4. Improve the UI

- Add transaction history
- Display APY estimates for liquidity providers
- Add token price charts using historical on-chain data
- Support wallet switching and multiple wallet providers

## Resources for Further Learning

- [Uniswap V2 Whitepaper](https://uniswap.org/whitepaper.pdf) — The original AMM design
- [Uniswap V2 Core Contracts](https://github.com/Uniswap/v2-core) — Production-grade AMM implementation
- [Avalanche Documentation](https://docs.avax.network/) — Learn more about building on Avalanche
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/) — Battle-tested smart contract libraries
- [Hardhat Documentation](https://hardhat.org/docs) — Advanced Hardhat usage

## Share Your Work!

We would love to see what you built! Share your project in the **#share-your-work** channel in the UNCHAIN Discord server.

If you have any questions or run into any issues, feel free to ask in the **#avalanche** channel.

## Final Checklist

Before you finish, make sure you have:

- [ ] Deployed both token contracts to Fuji testnet
- [ ] Deployed the AMM contract to Fuji testnet
- [ ] Verified your contracts on Snowtrace (optional but recommended)
- [ ] Tested providing liquidity through the UI
- [ ] Tested swapping tokens through the UI
- [ ] Tested withdrawing liquidity through the UI
- [ ] Shared your project with the community

Excellent work completing this project. Keep building! 🚀
