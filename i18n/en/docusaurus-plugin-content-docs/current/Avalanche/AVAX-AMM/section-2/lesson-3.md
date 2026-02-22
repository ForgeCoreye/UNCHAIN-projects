---
title: Implement the Swap Function
---

## Overview

In this lesson, we will implement the `swap` function, which is the core functionality of our AMM. This function enables users to exchange one token for another using the constant product formula.

## How Swapping Works

When a user swaps Token X for Token Y:

1. The user sends an amount of Token X to the pool
2. The pool calculates how much Token Y to return using the constant product formula
3. The pool's reserve of Token X increases and Token Y decreases
4. The product `x * y` remains constant (minus fees)

## Calculating the Swap Amount

Using the constant product formula `x * y = k`:

- Before swap: `x * y = k`
- After swap: `(x + dx) * (y - dy) = k`

Solving for `dy` (the amount of Token Y the user receives):

```
dy = (y * dx) / (x + dx)
```

Let's implement this as a view function first:

```solidity
/**
 * @dev Returns the amount of Token Y that will be received
 * in exchange for a given amount of Token X.
 */
function getSwapTokenXEstimate(
    uint256 amountX
) public view activePool returns (uint256 amountY) {
    uint256 tokenXAfter = totalAmount[_tokenX] + amountX;
    uint256 tokenYAfter = (totalAmount[_tokenX] * totalAmount[_tokenY]) /
        tokenXAfter;
    amountY = totalAmount[_tokenY] - tokenYAfter;

    // Ensure there is enough liquidity
    if (amountY == totalAmount[_tokenY]) {
        revert("AMM: Insufficient liquidity");
    }
}

/**
 * @dev Returns the amount of Token X that will be received
 * in exchange for a given amount of Token Y.
 */
function getSwapTokenYEstimate(
    uint256 amountY
) public view activePool returns (uint256 amountX) {
    uint256 tokenYAfter = totalAmount[_tokenY] + amountY;
    uint256 tokenXAfter = (totalAmount[_tokenX] * totalAmount[_tokenY]) /
        tokenYAfter;
    amountX = totalAmount[_tokenX] - tokenXAfter;

    // Ensure there is enough liquidity
    if (amountX == totalAmount[_tokenX]) {
        revert("AMM: Insufficient liquidity");
    }
}
```

## Implementing the Swap Functions

Now let's implement the actual swap functions:

```solidity
/**
 * @dev Swaps Token X for Token Y.
 * @param amountX The amount of Token X to swap.
 * @param minAmountY The minimum amount of Token Y expected (slippage protection).
 * @return amountY The amount of Token Y received.
 */
function swapTokenX(
    uint256 amountX,
    uint256 minAmountY
) external activePool returns (uint256 amountY) {
    require(amountX > 0, "AMM: Amount must be greater than 0");

    amountY = getSwapTokenXEstimate(amountX);
    require(
        amountY >= minAmountY,
        "AMM: Slippage exceeded — received less than minimum"
    );

    _tokenX.transferFrom(msg.sender, address(this), amountX);
    totalAmount[_tokenX] += amountX;
    totalAmount[_tokenY] -= amountY;
    _tokenY.transfer(msg.sender, amountY);
}

/**
 * @dev Swaps Token Y for Token X.
 * @param amountY The amount of Token Y to swap.
 * @param minAmountX The minimum amount of Token X expected (slippage protection).
 * @return amountX The amount of Token X received.
 */
function swapTokenY(
    uint256 amountY,
    uint256 minAmountX
) external activePool returns (uint256 amountX) {
    require(amountY > 0, "AMM: Amount must be greater than 0");

    amountX = getSwapTokenYEstimate(amountY);
    require(
        amountX >= minAmountX,
        "AMM: Slippage exceeded — received less than minimum"
    );

    _tokenY.transferFrom(msg.sender, address(this), amountY);
    totalAmount[_tokenY] += amountY;
    totalAmount[_tokenX] -= amountX;
    _tokenX.transfer(msg.sender, amountX);
}
```

## Slippage Protection

Notice that both swap functions include a `minAmount` parameter. This is a critical safety feature known as **slippage protection**.

Slippage occurs when the price changes between the time a transaction is submitted and when it is executed on-chain. This can happen because:

- Other transactions are processed before yours
- The block takes longer than expected to be mined
- A malicious actor manipulates the price (sandwich attack)

By specifying a minimum acceptable output, users protect themselves from receiving far less than expected.

## Testing the Swap Function

```typescript
it("Should swap Token X for Token Y", async function () {
  // First, provide liquidity
  const amountX = ethers.parseEther("100");
  const amountY = ethers.parseEther("200");
  await tokenX.connect(provider).approve(amm.target, amountX);
  await tokenY.connect(provider).approve(amm.target, amountY);
  await amm.connect(provider).provide(amountX, amountY);

  // Now swap
  const swapAmount = ethers.parseEther("10");
  const expectedOutput = await amm.getSwapTokenXEstimate(swapAmount);
  const minOutput = (expectedOutput * 99n) / 100n; // 1% slippage tolerance

  await tokenX.connect(user).approve(amm.target, swapAmount);
  const tx = await amm.connect(user).swapTokenX(swapAmount, minOutput);
  await tx.wait();

  // Verify the user received Token Y
  const userBalanceY = await tokenY.balanceOf(user.address);
  expect(userBalanceY).to.be.gte(minOutput);
});
```

## Summary

In this lesson, you:

1. Learned how the constant product formula determines swap prices
2. Implemented view functions to estimate swap outputs
3. Implemented `swapTokenX` and `swapTokenY` with slippage protection
4. Understood why slippage protection is critical in DeFi

In the next lesson, we will implement the `withdraw` function that allows liquidity providers to remove their funds from the pool.
