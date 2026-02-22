---
title: Build the AMM Contract
---

## Overview

In this lesson, we will start building the AMM (Automated Market Maker) smart contract. The AMM contract forms the core of our decentralized exchange, enabling users to swap tokens and provide liquidity without relying on a traditional order book.

## What is an AMM?

An Automated Market Maker (AMM) is a type of decentralized exchange protocol that uses a mathematical formula to price assets. Unlike traditional exchanges that use an order book to match buyers and sellers, AMMs use liquidity pools.

The most common formula used is the **constant product formula**:

```
x * y = k
```

Where:
- `x` is the reserve of token A
- `y` is the reserve of token B
- `k` is a constant

This formula ensures that the product of the two token reserves always remains constant, which determines the price of each token relative to the other.

## Setting Up the Contract

Create a new file called `AMM.sol` inside the `contracts` directory:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract AMM {
    IERC20 private _tokenX; // Token X in the liquidity pool
    IERC20 private _tokenY; // Token Y in the liquidity pool
    uint256 public totalShare; // Total share issued for the pool
    mapping(address => uint256) public share; // Share of each liquidity provider
    mapping(IERC20 => uint256) public totalAmount; // Total amount of each token in the pool

    uint256 public constant PRECISION = 1_000_000; // Precision for share calculations

    constructor(IERC20 tokenX, IERC20 tokenY) {
        _tokenX = tokenX;
        _tokenY = tokenY;
    }
}
```

## Contract Variables Explained

### Token References

```solidity
IERC20 private _tokenX;
IERC20 private _tokenY;
```

These variables store references to the two ERC20 tokens that will be traded in the pool. We use the `IERC20` interface to interact with any standard ERC20 token.

### Share Tracking

```solidity
uint256 public totalShare;
mapping(address => uint256) public share;
```

Shares represent a liquidity provider's proportional ownership of the pool. When you add liquidity, you receive shares. When you remove liquidity, you burn shares and receive your proportional amount of each token back.

### Pool Reserves

```solidity
mapping(IERC20 => uint256) public totalAmount;
```

This mapping tracks how much of each token is currently held in the pool. As swaps occur, these amounts change, which in turn changes the exchange rate.

### Precision Constant

```solidity
uint256 public constant PRECISION = 1_000_000;
```

Because Solidity does not support floating-point arithmetic, we use this precision factor to handle fractional calculations for share distribution.

## Adding a Validation Modifier

Let's add a modifier to restrict certain functions to only be callable when the pool has liquidity:

```solidity
modifier activePool() {
    require(totalShare > 0, "AMM: Zero liquidity in pool");
    _;
}
```

This modifier will be applied to functions like `swap` and `withdraw` that require the pool to already have liquidity.

## Summary

In this lesson, you:

1. Learned what an AMM is and how the constant product formula works
2. Set up the basic structure of the `AMM.sol` contract
3. Defined the key state variables for tracking tokens, shares, and reserves
4. Added a validation modifier to protect pool functions

In the next lesson, we will implement the `provide` function that allows users to add liquidity to the pool.
