---
title: Implement the Provide Function
---

## Overview

In this lesson, we will implement the `provide` function, which allows users to add liquidity to the AMM pool. Liquidity providers deposit equal values of both tokens and receive pool shares in return.

## How Providing Liquidity Works

When a user provides liquidity:

1. They deposit an amount of Token X and a corresponding amount of Token Y
2. The ratio of tokens must match the current pool ratio (after the first deposit)
3. They receive shares proportional to their contribution
4. These shares can later be redeemed to withdraw their liquidity plus any earned fees

## Implementing Helper Functions

Before implementing `provide`, let's add some helper functions:

### getEquivalentTokenAmount

This function calculates how much of Token Y is needed given an amount of Token X (and vice versa), based on the current pool ratio:

```solidity
/**
 * @dev Returns the equivalent amount of the other token
 * given one token amount, based on the current pool ratio.
 */
function getEquivalentTokenXAmount(
    uint256 amountY
) public view activePool returns (uint256) {
    return (totalAmount[_tokenX] * amountY) / totalAmount[_tokenY];
}

function getEquivalentTokenYAmount(
    uint256 amountX
) public view activePool returns (uint256) {
    return (totalAmount[_tokenY] * amountX) / totalAmount[_tokenX];
}
```

## Implementing the Provide Function

Now let's implement the main `provide` function:

```solidity
/**
 * @dev Adds liquidity to the pool.
 * @param amountX Amount of Token X to deposit.
 * @param amountY Amount of Token Y to deposit.
 * @return share_ The number of shares issued to the liquidity provider.
 */
function provide(
    uint256 amountX,
    uint256 amountY
) external returns (uint256 share_) {
    require(amountX > 0, "AMM: Amount of token X must be greater than 0");
    require(amountY > 0, "AMM: Amount of token Y must be greater than 0");

    if (totalShare == 0) {
        // Initial liquidity provision
        share_ = 100 * PRECISION;
    } else {
        // Subsequent liquidity provision: verify ratio and calculate shares
        uint256 shareX = (totalShare * amountX) / totalAmount[_tokenX];
        uint256 shareY = (totalShare * amountY) / totalAmount[_tokenY];
        require(
            shareX == shareY,
            "AMM: Provided amounts must be equivalent in value"
        );
        share_ = shareX;
    }

    require(share_ > 0, "AMM: Share amount too small");

    // Transfer tokens from the user to the contract
    _tokenX.transferFrom(msg.sender, address(this), amountX);
    _tokenY.transferFrom(msg.sender, address(this), amountY);

    // Update pool state
    totalAmount[_tokenX] += amountX;
    totalAmount[_tokenY] += amountY;
    totalShare += share_;
    share[msg.sender] += share_;
}
```

## Understanding the Logic

### First Liquidity Provider

When the pool is empty (`totalShare == 0`), the first liquidity provider sets the initial price ratio. They receive a fixed amount of shares (100 * PRECISION in our implementation). This is an arbitrary starting value.

### Subsequent Liquidity Providers

For subsequent providers, we calculate shares based on their contribution relative to the existing pool:

```
share = (totalShare * amountDeposited) / totalPoolAmount
```

We calculate this for both tokens and verify they are equal. If they are not equal, it means the user is not providing liquidity at the correct ratio, so the transaction reverts.

### Token Transfers

We use `transferFrom` to pull tokens from the user's wallet into the contract. This requires the user to have first called `approve` on both token contracts, granting the AMM contract permission to spend their tokens.

## Testing the Provide Function

Here's an example of how to test the `provide` function in your test file:

```typescript
it("Should provide liquidity and issue shares", async function () {
  const amountX = ethers.parseEther("100");
  const amountY = ethers.parseEther("200");

  // Approve token transfers
  await tokenX.connect(user).approve(amm.target, amountX);
  await tokenY.connect(user).approve(amm.target, amountY);

  // Provide liquidity
  const tx = await amm.connect(user).provide(amountX, amountY);
  await tx.wait();

  // Verify shares were issued
  const userShare = await amm.share(user.address);
  expect(userShare).to.be.gt(0);

  // Verify pool balances updated
  expect(await amm.totalAmount(tokenX.target)).to.equal(amountX);
  expect(await amm.totalAmount(tokenY.target)).to.equal(amountY);
});
```

## Summary

In this lesson, you:

1. Learned how liquidity provision works in an AMM
2. Implemented helper functions to calculate equivalent token amounts
3. Implemented the `provide` function with proper validation
4. Understood the difference between the first and subsequent liquidity providers

In the next lesson, we will implement the `swap` function that allows users to trade between tokens.
