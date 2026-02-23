# Building a React Minting App

Full guide for building a minting page with Next.js + RainbowKit + wagmi.

**Official example:** [examples/edition/rainbowkit-mint](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/rainbowkit-mint)

## Prerequisites

- Next.js 14+ (App Router)
- RainbowKit for wallet connection
- wagmi for React Ethereum hooks
- A deployed Manifold product (instance ID)

## 1. Project Setup

```bash
npx create-next-app@latest my-mint-page --typescript --tailwind --app
cd my-mint-page
npm install @manifoldxyz/client-sdk @rainbow-me/rainbowkit wagmi viem @tanstack/react-query
```

## 2. Wagmi + RainbowKit Config

Create `src/app/providers.tsx`:

```tsx
'use client';

import { WagmiProvider, createConfig, http } from 'wagmi';
import { mainnet, base, sepolia } from 'wagmi/chains';
import { RainbowKitProvider, getDefaultConfig } from '@rainbow-me/rainbowkit';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import '@rainbow-me/rainbowkit/styles.css';

const config = getDefaultConfig({
  appName: 'My Mint Page',
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID || '',
  chains: [mainnet, base, sepolia],
  transports: {
    [mainnet.id]: http(process.env.NEXT_PUBLIC_RPC_URL_MAINNET),
    [base.id]: http(process.env.NEXT_PUBLIC_RPC_URL_BASE),
    [sepolia.id]: http(process.env.NEXT_PUBLIC_RPC_URL_SEPOLIA),
  },
});

const queryClient = new QueryClient();

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider>{children}</RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

Wrap your layout in `src/app/layout.tsx`:

```tsx
import { Providers } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

## 3. MintButton Component

Create `src/components/MintButton.tsx`:

```tsx
'use client';

import { useState } from 'react';
import { useAccount, useConfig, useWalletClient } from 'wagmi';
import {
  createClient,
  createPublicProviderWagmi,
  createAccountViem,
  isEditionProduct,
  isBlindMintProduct,
} from '@manifoldxyz/client-sdk';
import type { PreparedPurchase, Product } from '@manifoldxyz/client-sdk';
import { ClientSDKError } from '@manifoldxyz/client-sdk';

interface MintButtonProps {
  instanceId: string;
  quantity?: number;
}

export default function MintButton({ instanceId, quantity = 1 }: MintButtonProps) {
  const { address, isConnected } = useAccount();
  const { data: walletClient } = useWalletClient();
  const config = useConfig();

  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [txHash, setTxHash] = useState<string | null>(null);

  const handleMint = async () => {
    if (!address || !walletClient) return;

    setLoading(true);
    setError(null);
    setTxHash(null);

    try {
      // 1. Create client
      const publicProvider = createPublicProviderWagmi({ config });
      const client = createClient({ publicProvider });

      // 2. Fetch product
      const product = await client.getProduct(instanceId);
      if (!isEditionProduct(product) && !isBlindMintProduct(product)) {
        throw new Error('Unsupported product type');
      }

      // 3. Check status
      const status = await product.getStatus();
      if (status !== 'active') {
        setError(`This drop is ${status}`);
        return;
      }

      // 4. Check eligibility
      const allocations = await product.getAllocations({ recipientAddress: address });
      if (!allocations.isEligible) {
        setError(allocations.reason || 'Not eligible to mint');
        return;
      }

      // 5. Create account adapter
      const account = createAccountViem({ walletClient });

      // 6. Prepare purchase
      const prepared = await product.preparePurchase({
        userAddress: address,
        payload: { quantity },
        account,
      });

      // 7. Execute purchase
      const result = await product.purchase({
        account,
        preparedPurchase: prepared,
      });

      setTxHash(result.transactionReceipt.txHash);
    } catch (err) {
      if (err instanceof ClientSDKError) {
        setError(err.message);
      } else {
        setError((err as Error).message || 'Mint failed');
      }
    } finally {
      setLoading(false);
    }
  };

  if (!isConnected) {
    return <p>Connect your wallet to mint</p>;
  }

  return (
    <div>
      <button onClick={handleMint} disabled={loading}>
        {loading ? 'Minting...' : `Mint ${quantity}`}
      </button>

      {error && <p style={{ color: 'red' }}>{error}</p>}
      {txHash && <p>Success! TX: {txHash}</p>}
    </div>
  );
}
```

## 4. Page

```tsx
'use client';

import { ConnectButton } from '@rainbow-me/rainbowkit';
import MintButton from '@/components/MintButton';

const INSTANCE_ID = process.env.NEXT_PUBLIC_INSTANCE_ID || '';

export default function Home() {
  return (
    <main>
      <h1>My NFT Drop</h1>
      <ConnectButton />
      <MintButton instanceId={INSTANCE_ID} quantity={1} />
    </main>
  );
}
```

## 5. Environment Variables

Create `.env.local`:

```env
NEXT_PUBLIC_INSTANCE_ID=your_instance_id
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id
NEXT_PUBLIC_RPC_URL_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
```

## 6. Displaying Product Info

```tsx
import { useEffect, useState } from 'react';

function ProductDisplay({ instanceId }: { instanceId: string }) {
  const config = useConfig();
  const [product, setProduct] = useState<any>(null);

  useEffect(() => {
    async function load() {
      const publicProvider = createPublicProviderWagmi({ config });
      const client = createClient({ publicProvider });
      const p = await client.getProduct(instanceId);
      setProduct(p);
    }
    load();
  }, [instanceId, config]);

  if (!product) return <p>Loading...</p>;

  const { publicData } = product.data;
  const preview = product.previewData;

  return (
    <div>
      {preview.thumbnail && <img src={preview.thumbnail} alt={preview.title} />}
      <h2>{preview.title}</h2>
      <p>{preview.description}</p>
    </div>
  );
}
```

## Step-by-Step Transaction UI

For products with ERC-20 pricing (multiple steps), see `references/transaction-steps.md` for the step-by-step modal pattern.

**Official example:** [examples/edition/step-by-step-mint](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/step-by-step-mint)

## Best Practices

- **Always check status** before showing the mint button
- **Show cost breakdown** from `prepared.cost` before confirming
- **Handle wallet disconnection** gracefully
- **Test on Sepolia** before deploying with mainnet instance IDs
- **Use `getAllocations`** to show remaining mints for allowlist drops
- **Disable the button** while transactions are in progress
