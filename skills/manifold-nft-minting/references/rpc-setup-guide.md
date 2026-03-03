# RPC Node Setup Guide

## What Is an RPC Node?

An RPC (Remote Procedure Call) node is your application's gateway to a blockchain network. It lets you read on-chain data (balances, contract state) and send transactions (mints, transfers). Public RPCs exist but are rate-limited and unreliable for production use.

## Why You Need a Dedicated RPC

- **Reliability** — Public RPCs throttle requests and can go down
- **Speed** — Dedicated nodes respond faster, critical for minting bots
- **Rate limits** — Public endpoints cap requests/second; dedicated plans offer much higher limits
- **Production requirement** — The Manifold SDK needs stable RPC access for preparing and executing purchases

## Setup with Alchemy (Recommended)

1. **Create an account** at [alchemy.com](https://www.alchemy.com/) (free tier available)
2. **Create a new app** — Click "Create new app" from the dashboard
3. **Select your network(s):**
   - Ethereum Mainnet — for production mints on Ethereum
   - Base Mainnet — for production mints on Base
   - Optimism Mainnet — for production mints on Optimism
   - Sepolia — for testing (always start here)
4. **Copy your RPC URL** — it looks like: `https://eth-mainnet.g.alchemy.com/v2/YOUR_API_KEY`
5. **Store in `.env`:**

```env
RPC_URL_MAINNET=https://eth-mainnet.g.alchemy.com/v2/your_key
RPC_URL_BASE=https://base-mainnet.g.alchemy.com/v2/your_key
RPC_URL_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/your_key
```

6. **Add `.env` to `.gitignore`** to prevent committing secrets:

```
# .gitignore
.env
.env.local
```

## Alternative Providers

| Provider | Free Tier | Supported Chains | Notes |
|----------|-----------|------------------|-------|
| [Alchemy](https://www.alchemy.com/) | Yes | ETH, Base, Optimism, Sepolia, + more | Most popular, generous free tier |
| [Infura](https://www.infura.io/) | Yes | ETH, Optimism, Sepolia, + more | Long-established provider |
| [QuickNode](https://www.quicknode.com/) | Yes | ETH, Base, Optimism, Sepolia, + more | Fast response times |

All providers follow the same pattern: sign up, create an endpoint, copy the URL.

## Using RPC URLs in Code

### Server-side (Node.js bot / script)

```typescript
import 'dotenv/config';
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';

const publicClient = createPublicClient({
  chain: mainnet,
  transport: http(process.env.RPC_URL_MAINNET!),
});
```

### Client-side (React / Next.js)

Use `NEXT_PUBLIC_` prefix for environment variables that need to be available in the browser:

```env
NEXT_PUBLIC_RPC_URL_MAINNET=https://eth-mainnet.g.alchemy.com/v2/your_key
```

```typescript
import { http } from 'wagmi';
import { mainnet } from 'wagmi/chains';

const transports = {
  [mainnet.id]: http(process.env.NEXT_PUBLIC_RPC_URL_MAINNET),
};
```

## Best Practices

- **Always use environment variables** — never hardcode RPC URLs in source code
- **Use separate keys per environment** — different API keys for dev vs production
- **Set up fallback endpoints** — configure a second provider as backup
- **Monitor rate limits** — check your provider dashboard for usage
- **Start with Sepolia** — test on the testnet before using mainnet RPCs
