---
title: Build the Frontend — Project Setup
---

## Overview

Now that our AMM smart contract is complete and tested, it's time to build a frontend so users can interact with it through a web browser. In this section, we will build a React-based frontend using Next.js.

## Project Structure

The frontend project is already scaffolded in the `frontend` directory. Let's take a look at the key files:

```
frontend/
├── components/
│   ├── Container.tsx      # Main container component
│   ├── InputBox.tsx       # Reusable input component
│   └── SelectTab.tsx      # Tab selector component
├── hooks/
│   ├── useContract.ts     # Hook for interacting with contracts
│   └── useWallet.ts       # Hook for wallet connection
├── pages/
│   ├── _app.tsx           # Next.js app entry point
│   └── index.tsx          # Main page
├── utils/
│   └── contracts.ts       # Contract addresses and ABIs
└── styles/
    └── globals.css        # Global styles
```

## Setting Up Environment Variables

After deploying your contracts, you will need to configure the frontend with the deployed contract addresses. Create a `.env.local` file in the `frontend` directory:

```bash
NEXT_PUBLIC_AMM_ADDRESS=0xYourAMMContractAddress
NEXT_PUBLIC_TOKEN_X_ADDRESS=0xYourTokenXAddress
NEXT_PUBLIC_TOKEN_Y_ADDRESS=0xYourTokenYAddress
```

> **Note:** In Next.js, environment variables prefixed with `NEXT_PUBLIC_` are exposed to the browser. Never put private keys or secrets in these variables.

## Installing Dependencies

Navigate to the `frontend` directory and install the dependencies:

```bash
cd frontend
npm install
```

Key dependencies include:

- **ethers.js** — For interacting with the Ethereum blockchain
- **Next.js** — The React framework
- **TypeScript** — For type safety

## Connecting to MetaMask

Let's implement the wallet connection hook. Create `hooks/useWallet.ts`:

```typescript
import { useState, useCallback } from "react";
import { ethers } from "ethers";

export const useWallet = () => {
  const [account, setAccount] = useState<string | undefined>(undefined);
  const [provider, setProvider] = useState<
    ethers.BrowserProvider | undefined
  >(undefined);

  const connect = useCallback(async () => {
    if (typeof window.ethereum === "undefined") {
      alert(
        "MetaMask is not installed. Please install MetaMask to use this app."
      );
      return;
    }

    try {
      const browserProvider = new ethers.BrowserProvider(window.ethereum);
      const accounts = await browserProvider.send("eth_requestAccounts", []);
      setAccount(accounts[0]);
      setProvider(browserProvider);
    } catch (error) {
      console.error("Failed to connect wallet:", error);
    }
  }, []);

  const disconnect = useCallback(() => {
    setAccount(undefined);
    setProvider(undefined);
  }, []);

  return { account, provider, connect, disconnect };
};
```

## Setting Up Contract Utilities

Create `utils/contracts.ts` to export contract addresses and ABIs:

```typescript
export const AMM_ADDRESS = process.env.NEXT_PUBLIC_AMM_ADDRESS as string;
export const TOKEN_X_ADDRESS = process.env.NEXT_PUBLIC_TOKEN_X_ADDRESS as string;
export const TOKEN_Y_ADDRESS = process.env.NEXT_PUBLIC_TOKEN_Y_ADDRESS as string;

// Import the generated ABI files from your Hardhat compilation
export { default as AmmAbi } from "../artifacts/contracts/AMM.sol/AMM.json";
export { default as ERC20Abi } from "../artifacts/contracts/ERC20Tokens.sol/USDCToken.json";
```

## Adding Avalanche Fuji Testnet to MetaMask

To test your application, you need to add the Avalanche Fuji C-Chain testnet to MetaMask:

| Setting | Value |
|---|---|
| Network Name | Avalanche Fuji Testnet |
| RPC URL | https://api.avax-test.network/ext/bc/C/rpc |
| Chain ID | 43113 |
| Currency Symbol | AVAX |
| Block Explorer | https://testnet.snowtrace.io/ |

You can add these settings manually in MetaMask under **Settings → Networks → Add Network**.

## Getting Testnet AVAX

To deploy contracts and pay for transactions on the Fuji testnet, you need testnet AVAX. You can get some from the official Avalanche faucet:

👉 [https://faucet.avax.network/](https://faucet.avax.network/)

Select **Fuji** as the network and enter your wallet address. You will receive testnet AVAX within a few seconds.

## Summary

In this lesson, you:

1. Explored the frontend project structure
2. Set up environment variables for contract addresses
3. Implemented the `useWallet` hook for MetaMask connection
4. Created contract utility exports
5. Configured MetaMask for the Avalanche Fuji testnet

In the next lesson, we will implement the contract interaction hook and start building the UI components.
