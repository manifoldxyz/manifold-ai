# Public Providers

Public providers handle read-only blockchain operations (balance queries, contract reads, gas estimation). Required for `createClient()`.

## Available Providers

| Provider | Import | Use With |
|----------|--------|----------|
| `createPublicProviderWagmi` | `@manifoldxyz/client-sdk/adapters` | Wagmi `Config` |
| `createPublicProviderViem` | `@manifoldxyz/client-sdk/adapters` | viem `PublicClient` map |
| `createPublicProviderEthers5` | `@manifoldxyz/client-sdk/adapters` | ethers v5 `Provider` map |

## Wagmi Provider (React Apps)

Best for React apps already using wagmi. Supports all chains in your wagmi config automatically.

```typescript
import { createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { mainnet, base, optimism } from '@wagmi/core/chains';

const config = createConfig({
  chains: [mainnet, base, optimism],
  transports: {
    [mainnet.id]: http('YOUR_MAINNET_RPC'),
    [base.id]: http('YOUR_BASE_RPC'),
    [optimism.id]: http('YOUR_OPTIMISM_RPC'),
  },
});

const publicProvider = createPublicProviderWagmi({ config });
```

### With React hooks

```typescript
import { useConfig } from 'wagmi';

function MintComponent() {
  const config = useConfig();
  const publicProvider = createPublicProviderWagmi({ config });
  const client = createClient({ publicProvider });
  // ...
}
```

## Viem Provider

Pass a `Record<chainId, PublicClient>`:

```typescript
import { createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet, base } from 'viem/chains';

const publicProvider = createPublicProviderViem({
  [mainnet.id]: createPublicClient({ chain: mainnet, transport: http('YOUR_MAINNET_RPC') }),
  [base.id]: createPublicClient({ chain: base, transport: http('YOUR_BASE_RPC') }),
});
```

### Single Chain

```typescript
const networkId = 8453; // Base
const publicClient = createPublicClient({ chain: base, transport: http(RPC_URL) });
const publicProvider = createPublicProviderViem({ [networkId]: publicClient });
```

## Ethers v5 Provider

Pass a `Record<chainId, Provider>`:

```typescript
import { createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const publicProvider = createPublicProviderEthers5({
  1: new ethers.providers.JsonRpcProvider('YOUR_MAINNET_RPC'),
  8453: new ethers.providers.JsonRpcProvider('YOUR_BASE_RPC'),
});
```

## IPublicProvider Interface

All providers implement:

```typescript
interface IPublicProvider {
  getBalance(params: {
    address: string;
    networkId: number;
    tokenAddress?: string;    // Omit for native token
  }): Promise<bigint>;

  estimateContractGas(params: {
    contractAddress: string;
    abi: readonly unknown[];
    functionName: string;
    args?: readonly unknown[];
    from: string;
    value?: bigint;
    networkId: number;
  }): Promise<bigint>;

  readContract<T = unknown>(params: {
    contractAddress: string;
    abi: readonly unknown[];
    functionName: string;
    args?: readonly unknown[];
    networkId: number;
  }): Promise<T>;

  subscribeToContractEvents(params: {
    contractAddress: string;
    abi: readonly unknown[];
    networkId: number;
    topics: string[];
    callback: (log: unknown) => void;
  }): Promise<() => void>;
}
```

## Choosing a Provider

| Scenario | Provider | Why |
|----------|---------|-----|
| React + wagmi | `createPublicProviderWagmi` | Uses existing wagmi config, automatic multi-chain |
| Server-side (modern) | `createPublicProviderViem` | Lightweight, good TypeScript support |
| Server-side (legacy) | `createPublicProviderEthers5` | If already using ethers v5 |
| Multi-chain app | Any — all support `Record<chainId, client>` | Map chain IDs to clients |

## RPC Providers

You need RPC endpoints for each chain you support. Options:

- [Alchemy](https://www.alchemy.com/) — most popular, free tier available
- [Infura](https://www.infura.io/) — Ethereum-focused
- [QuickNode](https://www.quicknode.com/) — multi-chain
- Public RPCs — free but rate-limited, not recommended for production
