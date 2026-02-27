# Adapters (Account + Public Provider)

The SDK separates read operations (Public Provider) from write operations (Account Adapter).

- **Public Provider** → blockchain reads (balances, contract state, gas estimation). Required for `createClient()`.
- **Account Adapter** → transaction signing and sending. Required for `purchase()`.

## Setup by Library

> **Recommendation:** Prefer **viem** for new projects. Use wagmi for React apps with wallet connection libraries. Only use ethers v5 for existing ethers codebases.

### Viem (recommended — server-side or custom setup)

```typescript
import { createClient, createPublicProviderViem, createAccountViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

// Provider: pass Record<chainId, PublicClient>
const publicProvider = createPublicProviderViem({
  [base.id]: createPublicClient({
    chain: base,
    transport: http('YOUR_RPC'), // or http() for public RPC
  }),
});
const client = createClient({ publicProvider });

// Account: from private key or browser wallet
const wallet = privateKeyToAccount('0xPRIVATE_KEY' as `0x${string}`);
const walletClient = createWalletClient({
  account: wallet,
  chain: base,
  transport: http('YOUR_RPC'), // or http() for public RPC
});
const account = createAccountViem({ walletClient });
```

### Wagmi (React apps with wallet connection libraries)

```typescript
import { createClient, createPublicProviderWagmi, createAccountViem } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { useConfig, useWalletClient } from 'wagmi';
import { mainnet, base } from '@wagmi/core/chains';

// Provider: use wagmi config
const config = useConfig(); // or createConfig({ ... })
const publicProvider = createPublicProviderWagmi({ config });
const client = createClient({ publicProvider });

// Account: wagmi uses viem under the hood
const { data: walletClient } = useWalletClient();
const account = createAccountViem({ walletClient });
```

**Full wagmi config example:**

```typescript
const config = createConfig({
  chains: [mainnet, base],
  transports: {
    [mainnet.id]: http('YOUR_MAINNET_RPC'), // or http() for public RPC
    [base.id]: http('YOUR_BASE_RPC'),       // or http() for public RPC
  },
});
```

### Ethers v5 (legacy)

```typescript
import { createClient, createPublicProviderEthers5, createAccountEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

// Provider: pass Record<chainId, Provider>
const provider = new ethers.providers.JsonRpcProvider('YOUR_RPC');
const publicProvider = createPublicProviderEthers5({ 1: provider });
const client = createClient({ publicProvider });

// Account: from private key or browser signer
const wallet = new ethers.Wallet('YOUR_PRIVATE_KEY', provider);
const account = createAccountEthers5({ wallet });
```

## Choosing an Adapter

| Scenario | Provider | Account |
|----------|---------|---------|
| Server bot / scripts (recommended) | `createPublicProviderViem` | `createAccountViem` + `privateKeyToAccount` |
| React + wagmi + wallet library | `createPublicProviderWagmi` | `createAccountViem` + `useWalletClient()` |
| Existing viem app | `createPublicProviderViem` | `createAccountViem` |
| Existing ethers v5 app | `createPublicProviderEthers5` | `createAccountEthers5` + `ethers.Wallet` |

## Multi-Chain Provider

All provider factories accept `Record<chainId, client>`:

```typescript
// Viem: multiple chains
const publicProvider = createPublicProviderViem({
  1: createPublicClient({ chain: mainnet, transport: http(MAINNET_RPC) }),
  8453: createPublicClient({ chain: base, transport: http(BASE_RPC) }),
  10: createPublicClient({ chain: optimism, transport: http(OPTIMISM_RPC) }),
});

// Ethers v5: multiple chains
const publicProvider = createPublicProviderEthers5({
  1: new ethers.providers.JsonRpcProvider(MAINNET_RPC),
  8453: new ethers.providers.JsonRpcProvider(BASE_RPC),
});
```

## IAccount Interface

```typescript
interface IAccount {
  readonly adapterType: 'ethers5' | 'ethers6' | 'viem' | 'wagmi';
  getAddress(): Promise<string>;
  sendTransaction(request: UniversalTransactionRequest): Promise<string>;
  sendTransactionWithConfirmation(
    request: UniversalTransactionRequest,
    options?: { confirmations?: number }
  ): Promise<UniversalTransactionResponse>;
  getBalance(networkId: number, tokenAddress?: string): Promise<Money>;
  switchNetwork(chainId: number): Promise<void>;
  signMessage(message: string): Promise<string>;
}
```

## IPublicProvider Interface

```typescript
interface IPublicProvider {
  getBalance(params: { address: string; networkId: number; tokenAddress?: string }): Promise<bigint>;
  estimateContractGas(params: {
    contractAddress: string; abi: readonly unknown[];
    functionName: string; args?: readonly unknown[];
    from: string; value?: bigint; networkId: number;
  }): Promise<bigint>;
  readContract<T>(params: {
    contractAddress: string; abi: readonly unknown[];
    functionName: string; args?: readonly unknown[];
    networkId: number;
  }): Promise<T>;
  subscribeToContractEvents(params: {
    contractAddress: string; abi: readonly unknown[];
    networkId: number; topics: string[];
    callback: (log: unknown) => void;
  }): Promise<() => void>;
}
```
