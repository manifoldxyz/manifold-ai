# Getting Started with Manifold Client SDK

## Overview

The [Manifold Client SDK](https://github.com/manifoldxyz/client-sdk) (`@manifoldxyz/client-sdk`) enables building custom NFT minting experiences for [Manifold Studio](https://studio.manifold.xyz/) products. No API keys required.

**SDK Docs:** https://docs.manifold.xyz/client-sdk/

## Requirements

- **Node.js 18.0.0+**
- A package manager (npm, pnpm, or yarn)
- [RPC providers](https://www.alchemy.com/overviews/private-rpc-endpoint) for the networks you support

## Installation

```bash
npm install @manifoldxyz/client-sdk
```

## Client Initialization

The SDK entry point is `createClient()`. It requires a `publicProvider` for blockchain read operations.

```typescript
import { createClient } from '@manifoldxyz/client-sdk';

const client = createClient({ publicProvider });
```

### Provider Setup (pick one)

> **Recommendation:** Prefer **viem** for new projects. Use wagmi if building a React app with RainbowKit or similar wallet UI. Only use ethers v5 if integrating into an existing ethers codebase.

#### With Viem (recommended)

```typescript
import 'dotenv/config';
import { createClient, createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';

const publicClient = createPublicClient({
  chain: mainnet,
  transport: http(process.env.RPC_URL!),
  // Or use public RPC (rate-limited): transport: http()
});

const publicProvider = createPublicProviderViem({ 1: publicClient });
const client = createClient({ publicProvider });
```

#### With Wagmi (React apps)

> **CRITICAL: When using RainbowKit, ALWAYS install `wagmi@^2.9.0`.** Do NOT run `npm install wagmi` or `npm install wagmi@latest` — this will install wagmi 3.x, which is incompatible with RainbowKit. The correct command is: `npm install wagmi@^2.9.0`

```typescript
import { createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { mainnet, base } from '@wagmi/core/chains';

const config = createConfig({
  chains: [mainnet, base],
  transports: {
    [mainnet.id]: http(process.env.RPC_URL_MAINNET!),
    [base.id]: http(process.env.RPC_URL_BASE!),
    // Or use public RPC (rate-limited): http()
  },
});

const publicProvider = createPublicProviderWagmi({ config });
const client = createClient({ publicProvider });
```

#### With Ethers v5 (legacy)

```typescript
import { createClient, createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL!);
const publicProvider = createPublicProviderEthers5({ 1: provider });
const client = createClient({ publicProvider });
```

## Fetching a Product

```typescript
const product = await client.getProduct('4150231280');
// Or use a full URL:
const product = await client.getProduct('https://manifold.xyz/@creator/id/4150231280');
```

## Quick Start (Full Example)

```typescript
import {
  createClient,
  createPublicProviderWagmi,
  createAccountViem,
  EditionProduct,
} from '@manifoldxyz/client-sdk';
import { createConfig, http, getAccount, getWalletClient } from '@wagmi/core';
import { mainnet } from '@wagmi/core/chains';

const config = createConfig({
  chains: [mainnet],
  transports: { [mainnet.id]: http(process.env.RPC_URL!) },
});

const client = createClient({
  publicProvider: createPublicProviderWagmi({ config }),
});

const product = (await client.getProduct('4150231280')) as EditionProduct;

const account = getAccount(config);
if (!account.address) throw new Error('No wallet connected');

const prepared = await product.preparePurchase({
  userAddress: account.address,
  payload: { quantity: 1 },
});

const walletClient = await getWalletClient(config);
const order = await product.purchase({
  account: createAccountViem({ walletClient }),
  preparedPurchase: prepared,
});

console.log(`Minted! TX: ${order.transactionReceipt.txHash}`);
```
