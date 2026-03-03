# Supported Networks

## Chain IDs

| Network | Chain ID | Type | Notes |
|---------|----------|------|-------|
| Ethereum | 1 | Mainnet | Primary chain |
| Optimism | 10 | L2 | Lower gas fees |
| Base | 8453 | L2 | Lower gas fees |
| Shape | 360 | L2 | Manifold-specific |
| ApeChain | 33139 | L2 | APE ecosystem |
| Sepolia | 11155111 | Testnet | For development/testing |

## Multi-Chain Provider Setup

> **Recommendation:** Prefer **viem** for new projects. Use wagmi if building a React app with a wallet connection library. Only use ethers v5 for existing ethers codebases.

### Viem (recommended)

```typescript
import { createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet, base, optimism, sepolia } from 'viem/chains';

// With dedicated RPC URLs (recommended for production)
const publicProvider = createPublicProviderViem({
  [mainnet.id]: createPublicClient({ chain: mainnet, transport: http(MAINNET_RPC) }),
  [base.id]: createPublicClient({ chain: base, transport: http(BASE_RPC) }),
  [optimism.id]: createPublicClient({ chain: optimism, transport: http(OPTIMISM_RPC) }),
  [sepolia.id]: createPublicClient({ chain: sepolia, transport: http(SEPOLIA_RPC) }),
});

// With public RPC fallback (rate-limited, OK for development)
const publicProviderPublic = createPublicProviderViem({
  [mainnet.id]: createPublicClient({ chain: mainnet, transport: http() }),
  [base.id]: createPublicClient({ chain: base, transport: http() }),
  [sepolia.id]: createPublicClient({ chain: sepolia, transport: http() }),
});
```

### Wagmi (React apps)

> **CRITICAL: When using RainbowKit, ALWAYS install `wagmi@^2.9.0`.** Do NOT run `npm install wagmi` or `npm install wagmi@latest` — this will install wagmi 3.x, which is incompatible with RainbowKit. The correct command is: `npm install wagmi@^2.9.0`

```typescript
import { createConfig, http } from '@wagmi/core';
import { mainnet, base, optimism, sepolia } from '@wagmi/core/chains';

const config = createConfig({
  chains: [mainnet, base, optimism, sepolia],
  transports: {
    [mainnet.id]: http(MAINNET_RPC),   // or http() for public RPC
    [base.id]: http(BASE_RPC),         // or http() for public RPC
    [optimism.id]: http(OPTIMISM_RPC), // or http() for public RPC
    [sepolia.id]: http(SEPOLIA_RPC),   // or http() for public RPC
  },
});
```

### Ethers v5 (legacy)

```typescript
import { createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const publicProvider = createPublicProviderEthers5({
  1: new ethers.providers.JsonRpcProvider(MAINNET_RPC),
  8453: new ethers.providers.JsonRpcProvider(BASE_RPC),
  10: new ethers.providers.JsonRpcProvider(OPTIMISM_RPC),
  11155111: new ethers.providers.JsonRpcProvider(SEPOLIA_RPC),
});
```

## Shape and ApeChain (Custom Chains)

Shape (360) and ApeChain (33139) may not be available in viem/wagmi's built-in chain definitions. Define them manually:

```typescript
import { defineChain } from 'viem';

const shape = defineChain({
  id: 360,
  name: 'Shape',
  nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
  rpcUrls: {
    default: { http: ['https://mainnet.shape.network'] },
  },
  blockExplorers: {
    default: { name: 'Shape Explorer', url: 'https://shapescan.xyz' },
  },
});

const apechain = defineChain({
  id: 33139,
  name: 'ApeChain',
  nativeCurrency: { name: 'APE', symbol: 'APE', decimals: 18 },
  rpcUrls: {
    default: { http: ['https://rpc.apechain.com'] },
  },
  blockExplorers: {
    default: { name: 'ApeChain Explorer', url: 'https://apescan.io' },
  },
});
```

## Environment Variables Pattern

```env
# RPC URLs per network
RPC_URL_MAINNET=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
RPC_URL_BASE=https://base-mainnet.g.alchemy.com/v2/YOUR_KEY
RPC_URL_OPTIMISM=https://opt-mainnet.g.alchemy.com/v2/YOUR_KEY
RPC_URL_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY

# Target network (for single-chain apps)
NETWORK_ID=8453
```

## Testing on Sepolia

Always test on Sepolia first:

1. Get Sepolia ETH from a faucet (e.g., https://sepoliafaucet.com/)
2. Create a test product on [Manifold Studio](https://studio.manifold.xyz) using Sepolia
3. Set your network ID to `11155111`
4. Use a Sepolia RPC URL

```typescript
const publicProvider = createPublicProviderViem({
  11155111: createPublicClient({
    chain: sepolia,
    transport: http(process.env.RPC_URL_SEPOLIA!),
  }),
});
```
