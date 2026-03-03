# React Minting App (RainbowKit)

> **Note:** This reference assumes RainbowKit for wallet connection. If the user chose a different wallet library, adapt the wallet connection parts (providers, ConnectButton, wagmi config) to match their library's patterns. The Manifold SDK integration (MintButton, product queries) remains the same regardless of wallet library.

Build a minting page with Next.js + RainbowKit + wagmi.

**Official example:** [examples/edition/rainbowkit-mint](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/rainbowkit-mint)

## Dependencies

> **CRITICAL: ALWAYS install `wagmi@^2.9.0` — NEVER run `npm install wagmi` without the version pin.** Running `npm install wagmi` or `npm install wagmi@latest` will install wagmi 3.x, which is incompatible with RainbowKit. The correct command is shown below.

```bash
npm install @manifoldxyz/client-sdk @rainbow-me/rainbowkit wagmi@^2.9.0 viem @tanstack/react-query
```

## Setup

1. Follow [RainbowKit installation](https://rainbowkit.com/docs/installation) for wagmi config + providers
2. Render `<ConnectButton />` on the page
3. Create a MintButton component (below)

## Environment Variables

```env
NEXT_PUBLIC_INSTANCE_ID=your_instance_id
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
NEXT_PUBLIC_RPC_URL_SEPOLIA=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
```

## MintButton Component

Core pattern — all SDK interaction happens here:

```tsx
'use client';

import { useState } from 'react';
import { useAccount, useConfig, useWalletClient } from 'wagmi';
import {
  createClient, createPublicProviderWagmi, createAccountViem,
  isEditionProduct, isBlindMintProduct, ClientSDKError,
} from '@manifoldxyz/client-sdk';

export default function MintButton({ instanceId, quantity = 1 }: {
  instanceId: string; quantity?: number;
}) {
  const { address, isConnected } = useAccount();
  const { data: walletClient } = useWalletClient();
  const config = useConfig();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [txHash, setTxHash] = useState<string | null>(null);

  const handleMint = async () => {
    if (!address || !walletClient) return;
    setLoading(true); setError(null); setTxHash(null);

    try {
      const client = createClient({
        publicProvider: createPublicProviderWagmi({ config }),
      });

      const product = await client.getProduct(instanceId);
      if (!isEditionProduct(product) && !isBlindMintProduct(product)) {
        throw new Error('Unsupported product type');
      }

      const status = await product.getStatus();
      if (status !== 'active') { setError(`Drop is ${status}`); return; }

      const alloc = await product.getAllocations({ recipientAddress: address });
      if (!alloc.isEligible) { setError(alloc.reason || 'Not eligible'); return; }

      const account = createAccountViem({ walletClient });
      const prepared = await product.preparePurchase({
        userAddress: address, payload: { quantity }, account,
      });

      const result = await product.purchase({ account, preparedPurchase: prepared });
      setTxHash(result.transactionReceipt.txHash);
    } catch (err) {
      setError(err instanceof ClientSDKError ? err.message : (err as Error).message);
    } finally {
      setLoading(false);
    }
  };

  if (!isConnected) return <p>Connect wallet to mint</p>;

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

## Displaying Product Info

```tsx
const product = await client.getProduct(instanceId);
const onchain = await product.fetchOnchainData();
const media = await product.getPreviewMedia();
const inventory = await product.getInventory();

// Off-chain data
product.previewData.title
product.previewData.description
product.previewData.thumbnail

// On-chain data
onchain.cost.formatted           // "0.05 ETH"
inventory.totalPurchased          // minted count
inventory.totalSupply             // -1 = unlimited
```

## Step-by-Step Transaction UI

For ERC-20 priced products (multiple transaction steps), see `references/transaction-steps.md`.

**Official example:** [examples/edition/step-by-step-mint](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/step-by-step-mint)
