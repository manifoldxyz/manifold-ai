# Manifold Client SDK — Full Documentation

> This is a complete mirror of the official SDK documentation at https://docs.manifold.xyz/client-sdk/
> Source: https://manifold-1.gitbook.io/manifold-client-sdk/llms-full.txt
> **Use this as a fallback only when the focused reference files don't cover your needs.**

## Table of Contents (grep these headings to jump to sections)

- `# Why Manifold Client SDK?` — Overview and value prop
- `# Getting Started` — Installation, quick start, first product
- `# Creating a React minting app` — RainbowKit + wagmi pattern
- `# Advanced use cases` — Step-by-step transactions, ERC-20 pricing
- `# Creating a minting bot` — Server-side headless minting
- `# FAQ` — Supported networks, allowlists, error handling
- `# [for AI agents and LLMS] Checklist` — LLM-specific guidelines
- `# Release Notes` — SDK version history
- `# Manifold Client` — createClient() API
- `# getProduct` — Fetching products by ID/URL
- `# Product` — Product object structure
- `# Common` — Shared methods (purchase, getStatus, getAllocations, etc.)
- `# Edition Product` — Edition-specific API
- `# Blind Mint` — BlindMint-specific API
- `# Public Provider Adapters` — Wagmi, Viem, Ethers v5 providers
- `# Account Adapters` — Viem, Ethers v5 account adapters
- `# Transaction Steps` — Multi-step execution

---

# Why Manifold Client SDK?

The **Manifold Client SDK** exists to enable builders to create web3 apps, where [the creator is the platform](https://manifoldxyz.substack.com/p/manifold-creator).

This toolkit complements [**Manifold Studio**](https://studio.manifold.xyz/) such that anyone can build:

* **Custom storefronts** – Fetch products, check allocations, verify allowlists, and more.
* **Trusted checkout flows** – Secure web3 transactions with Manifold’s audited smart contracts.
* **Wallet agnostic UIs** – Official support for `ethers v5` and `viem`, with more adapters coming.

To learn more about how to build your own app and seamlessly integrate onchain purchases, see [**Getting Started**](https://manifoldxyz.gitbook.io/manifold-client-sdk/getting-started).

If you have any questions or feedback, see our [**FAQ**](https://manifoldxyz.gitbook.io/manifold-client-sdk/guides/faq) or join our [**Community Forum**](https://forum.manifold.xyz).


# Getting Started

## Overview

[Manifold Studio](https://studio.manifold.xyz/) enables the publishing of [**Edition**](https://help.manifold.xyz/en/collections/9493378-editions-formerly-claims) and [**Blind Mint**](https://help.manifold.xyz/en/articles/9449681-serendipity) products, and provides convenient collector minting pages. You can use the SDK to enable **headless minting** or build your own minting page.

## Requirements

Before getting started, make sure you have the following:

* **Node.js 18.0.0 or higher**
  * Check your version: `node --version`
  * Download from [nodejs.org](https://nodejs.org/)
* A package manager (npm, pnpm, or yarn)
* [**RPC providers**](https://www.alchemy.com/overviews/private-rpc-endpoint) for the networks you plan to support

## Installation <a href="#installation" id="installation"></a>

Install the SDK in your project:

```bash
npm install @manifoldxyz/client-sdk
```

#### Quick Start <a href="#quick-start" id="quick-start"></a>

{% tabs %}
{% tab title="index.ts" %}

```typescript
import { createClient, EditionProduct, createPublicProviderWagmi, createAccountViem } from '@manifoldxyz/client-sdk';
import { createConfig, http, getAccount, getWalletClient } from '@wagmi/core';
import { mainnet } from '@wagmi/core/chains';

// Create Wagmi config
const config = createConfig({
  chains: [mainnet],
  transports: {
    [mainnet.id]: http('YOUR_RPC_URL'), // or http() for public RPC
  },
});

// Initialize the Manifold client
const client = createClient({ publicProvider:createPublicProviderWagmi({ config }) });

// Fetch product
const product = await client.getProduct('4150231280') as EditionProduct;

// Get connected account from Wagmi
const account = getAccount(config);
if (!account.address) throw new Error('No wallet connected');

// Prepare purchase
const prepared = await product.preparePurchase({
  address: account.address,
  payload: { quantity: 1 },
});

// Get wallet client and create account adapter
const walletClient = await getWalletClient(config);

// Execute purchase
const order = await product.purchase({
  account: createAccountViem({ walletClient }),
  preparedPurchase: prepared,
});
const txHash = order.receipts[0]?.txHash;
console.log(`Edition purchase transaction: ${txHash}`);
```

{% endtab %}
{% endtabs %}

## Create your first product

Head over to [Manifold Studio](https://studio.manifold.xyz/) to [create your first product](https://help.manifold.xyz/en/collections/9493376-your-create-menu). The SDK currently supports [Edition](https://help.manifold.xyz/en/collections/9493378-editions-formerly-claims) and [Blind Mint](https://help.manifold.xyz/en/articles/9449681-serendipity) products, with more product types planned for the future.

1. Visit [studio.manifold.xyz](https://studio.manifold.xyz/) and sign in.
2. Navigate to the [Create+ Menu](https://help.manifold.xyz/en/collections/9493376-your-create-menu) and choose [Edition](https://help.manifold.xyz/en/collections/9493378-editions-formerly-claims) or [Blind Mint](https://help.manifold.xyz/en/articles/9449681-serendipity) .
3. Configure your asset, allowlists, supply caps, and price.
4. Publish the product and note the instance ID from the URL.
5. Use the instance ID or shareable URL with [`getProduct`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client/getproduct).

### Troubleshooting

#### Node.js Version Issues

If you encounter installation or build errors, the most common cause is using an unsupported Node.js version.

**Common error messages:**

* `npm ERR! engine Unsupported platform`
* `npm ERR! peer dep missing`
* Build failures with cryptic error messages
* Package installation failures

**Solution:**

1. Check your Node.js version:

   ```
   node --version
   ```
2. If you're using Node.js < 18.0.0, update to the latest LTS version:
   * Visit [nodejs.org](https://nodejs.org/) to download the latest LTS
   * Or use a version manager like `nvm`:

     ```
     nvm install --ltsnvm use --lts
     ```

[If you're an AI agent or LLM, please read this.](https://manifoldxyz.gitbook.io/manifold-client-sdk/guides/for-ai-agents-and-llms-checklist-and-guidelines)

### Building with AI

\
This entire site is converted into a single markdown doc that can fit inside the context window of most LLMs. See [The /llms.txt file](https://manifold-1.gitbook.io/manifold-client-sdk/llms-full.txt).

\\


# Creating a React minting app

{% hint style="info" %}
This example uses [RainbowKit](https://rainbowkit.com/) and [Viem adapter](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/account-adapters/viem). The SDK also works with Ethers v5 via the [EthersV5 adapter](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/account-adapters/ethersv5).
{% endhint %}

The repository includes a complete example at [examples/rainbowkit-mint](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/rainbowkit-mint), demonstrating how to implement minting with Manifold products including [Edition Products](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editionproduct)

**Overview**

The edition RainbowKit example showcases how to:

* Connect wallets with RainbowKit + wagmi
* Mint [Edition products](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editionproduct) through the Manifold Client SDK
* Run on Next.js 14 with the App Router and TypeScript
* Display mint progress, costs, and errors in the UI
* Complete example at [examples/rainbowkit-mint](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/rainbowkit-mint)

**Quick start**

1. **Install workspace dependencies**

   ```bash
   pnpm install
   ```
2. **Create environment variables**

   ```bash
   cp .env.example \
      env.local
   ```

   Fill in:

   ```env
   NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id
   NEXT_PUBLIC_INSTANCE_ID=your_edition_instance_id
   NEXT_PUBLIC_RPC_URL_SEPOLIA=your_alchemy_rpc_url
   ```

   `NEXT_PUBLIC_INSTANCE_ID` must point to an Edition product you published in Manifold Studio.\
   `NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID` is optional but required if you want to support [WalletConnect](https://dashboard.reown.com/sign-in) wallets.\
   `NEXT_PUBLIC_RPC_URL_SEPOLIA` your[ Alchemy Sepolia RPC URL](https://www.alchemy.com/overviews/private-rpc-endpoint).
3. **Launch the example**

```bash
pnpm dev
```

Visit `http://localhost:3000` and connect a wallet with RainbowKit’s `ConnectButton`.

**Key implementation steps**

1. Follow [RainbowKit setup instructions](https://rainbowkit.com/docs/installation)

Ensure you have the [ConnectButton](https://rainbowkit.com/docs/connect-button) component on your page.\
This handles wallet connections, which are required to create an [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) that the SDK uses for checks and transaction execution.

```typescript
'use client';

import { ConnectButton } from '@rainbow-me/rainbowkit';

export default function Home() {
  return (
    <main>
      <div>
        <ConnectButton />
      </div>
    </main>
  );
}
```

2. Implement Minting Logic in [MintButton.tsx](https://github.com/manifoldxyz/client-sdk/blob/main/packages/examples/edition/rainbowkit-mint/src/components/MintButton.tsx)

```typescript
'use client';

import { ConnectButton } from '@rainbow-me/rainbowkit';
import { MintButton } from '@/components/MintButton';

export default function Home() {
  return (
    <main>
      <h1>
        Manifold SDK + RainbowKit
      </h1>

      <div>
        <ConnectButton />
        <MintButton />
      </div>
    </main>
  );
}
```

**Core Steps**

a. Create a [Manifold Client](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client) with a public provider

```typescript
import { createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { useConfig } from 'wagmi';

// Get the Wagmi config from your React context
const config = useConfig();

// Create a public provider using Wagmi config
const publicProvider = createPublicProviderWagmi({ config });

// Initialize the client
const client = createClient({ publicProvider });
```

b. Create an [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) representing the connected user.

```typescript
import { createAccountViem } from '@manifoldxyz/client-sdk';
import { useWalletClient } from 'wagmi';

// Get the wallet client from wagmi
const { data: walletClient } = useWalletClient();

// Create the account adapter
const account = createAccountViem({
  walletClient,
});
```

c. Fetch the product and verify its type

```typescript
const product = await client.getProduct(INSTANCE_ID) as EditionProduct;
```

d. Check the product status to ensure it’s still active

```typescript
const productStatus = await product.getStatus();
if (productStatus !== 'active') {
  throw new Error(`Product is ${productStatus}`);
}
```

e. Prepare the purchase by specifying the amount

```typescript
const preparedPurchase = await product.preparePurchase({
  address: address,
  payload: {
    quantity: 1,
  },
});
```

f. Execute the purchase

```typescript
const order = await product.purchase({
  account,
  preparedPurchase,
});
```

Key points:

* `createAccountViem` wraps wagmi’s wallet client so the SDK can sign and send transactions on the user’s behalf.
* `preparePurchase` performs all eligibility checks (allowlists, supply, promo codes) and returns the total cost breakdown. Supply the same `account` so balance checks run against the connected wallet.
* `purchase` executes the transaction sequence (ERC-20 approvals, mint, etc.) and returns a [Receipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/receipt) with the final transaction hash and minted tokens.

**Display token media and on-chain stats**

You can enrich the UI with product art and live supply data directly from the SDK:

```typescript
const product = await client.getProduct(instanceId);

// Off-chain media and metadata (safe to render immediately)
const { asset, title, contract } = product.data.publicData;
const imageUrl = asset.image ?? asset.imagePreview;
const animationUrl = asset.animation ?? asset.animationPreview;

// Fetch on-chain data once (cost, supply, timing)
const onchainData = await product.fetchOnchainData();
const { totalMinted, totalSupply, startDate, endDate, cost } = onchainData;

return (
  <section>
    {imageUrl && <img src={imageUrl} alt={title} />}
    {animationUrl && (
      <video src={animationUrl} autoPlay loop muted playsInline />
    )}

    <dl>
      <dt>Price</dt>
      <dd>{cost.formatted}</dd>
      <dt>Minted</dt>
      <dd>{totalMinted}</dd>
      <dt>Total supply</dt>
      <dd>{totalSupply === -1 ? 'Unlimited' : totalSupply}</dd>
      <dt>Start date</dt>
      <dd>{startDate?.toLocaleString() ?? 'TBD'}</dd>
      <dt>End date</dt>
      <dd>{endDate?.toLocaleString() ?? 'Open'}</dd>
      <dt>Contract</dt>
      <dd>{contract.address}</dd>
    </dl>
  </section>
);
```

Best practices:

* **Check status** with [getStatus](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getstatus) before attempting a purchase to verify the product is active.
* **Handle** [**ClientSDKError**](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror) **codes** for common cases such as ineligibility, sold-out items, or insufficient funds.
* Call [`getAllocations`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getallocations) when you need to show remaining allowlist spots.
* Inspect [`Receipt.order`](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/order) to display which tokens were minted after `purchase`.


# Advanced use cases

We strongly recommend following this tutorial if you plan to support an [Edition](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editionproduct) product with any of the following configurations:

* Price set using an ERC-20 token

As discussed in [**Transaction Steps**](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactionstep), purchasing a product might require more than one on-chain transaction. We recommend executing each transaction explicitly (e.g., via button clicks).

This tutorial demonstrates how to handle the above using the SDK.

The repo ships with a full example at [examples/step-by-step-mint](https://github.com/manifoldxyz/client-sdk/tree/main/examples/step-by-step-mint), showing how to implement minting with a [Blind Mint](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/blind-mint) product.

**Install dependencies**

```bash
cd examples/step-by-step-mint
npm install
```

**Configure environment** – Copy `.env.example` to `.env` and set the following:

```bash
NEXT_PUBLIC_INSTANCE_ID= # You blind mint instance ID from Manifold Studio 
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID= # Get one at https://dashboard.reown.com/sign-in
```

**Run locally**

```bash
npm run dev
```

**Key implementation steps**

1. Follow [RainbowKit setup instructions](https://rainbowkit.com/docs/installation)

Make sure you render [ConnectButton](https://rainbowkit.com/docs/connect-button) on the page. This handles the user’s wallet connection, which is required to create an [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) that the SDK uses to perform checks and execute transactions later.

```typescript
'use client';

import { ConnectButton } from '@rainbow-me/rainbowkit';

export default function Home() {
  return (
    <main>
      <div>
        <ConnectButton />
      </div>
    </main>
  );
}
```

3. Implement the [MintButton.tsx](https://github.com/manifoldxyz/client-sdk/blob/main/examples/step-by-step-mint/src/components/MintButton.tsx)

On button click, initialize the client, fetch the product, and call `preparePurchase` to get the steps.

This is handled within [handlePreparePurchase](https://github.com/manifoldxyz/client-sdk/blob/3131d4374ad98b48ba1d17cc1a29e58428ebd121/examples/step-by-step-mint/src/components/MintButton.tsx#L28).

a. Create a [Manifold Client](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client)

```typescript
import { createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { useConfig } from 'wagmi';

// Get Wagmi config from React context
const config = useConfig();

// Create public provider and client
const publicProvider = createPublicProviderWagmi({ config });
const client = createClient({ publicProvider });
```

b. Fetch the product and validate the type

```typescript
const product = await client.getProduct(INSTANCE_ID);
if (!isBlindMintProduct(product)) {
  throw new Error('Is not a blind mint instance')
}
```

c. Check the product status to ensure it’s still active

```typescript
const productStatus = await product.getStatus();
if (productStatus !== 'active') {
  throw new Error(`Product is ${productStatus}`);
}
```

e. Prepare the purchase by specifying the purchase amount and capture the returned [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase)

```typescript
const preparedPurchase = await product.preparePurchase({
  address: address,
  payload: {
    quantity
  }
});
setPreparedPurchase(prepared)
```

Implement a function responsible for executing an individual step. This is handled in [handleExecuteStep](https://github.com/manifoldxyz/client-sdk/blob/3131d4374ad98b48ba1d17cc1a29e58428ebd121/examples/step-by-step-mint/src/components/MintButton.tsx#L70)

a. Create an [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) representing the connected user.

<pre class="language-tsx"><code class="lang-tsx">import { createAccountViem } from '@manifoldxyz/client-sdk'
<strong>export default function MintButton({ instanceId, quantity = 1 }: MintButtonButtonProps) {
</strong>  const { data: walletClient } = useWalletClient()
  //...other codes
  
  const handlePreparePurchase = async () => {
    const account = createAccountViem({
      walletClient: walletClient
    })
    //...other codes
  }
}
  
</code></pre>

b. Execute the step by calling the [execute](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute) function on each step

<pre class="language-tsx"><code class="lang-tsx">import { createAccountViem } from '@manifoldxyz/client-sdk'
<strong>export default function MintButton({ instanceId, quantity = 1 }: MintButtonButtonProps) {
</strong>  const { data: walletClient } = useWalletClient()
  //...other codes
  
  const handlePreparePurchase = async (stepIndex: number) => {
   //...other codes
   const step = preparedPurchase.steps[stepIndex]
   if (!step) {
    setError('Invalid step index')
    return
   }
   try {
    const result = await step.execute(account)
   catch (error) {
    setError(err.message || 'Transaction failed')
   }
   //...other codes
  }
}
  
</code></pre>

c. Render the [StepModal](https://github.com/manifoldxyz/client-sdk/blob/main/examples/step-by-step-mint/src/components/StepModal.tsx)

```tsx
  {preparedPurchase && (
      <StepModal
        isOpen={isModalOpen}
        onClose={handleCloseModal}
        steps={preparedPurchase.steps}
        onExecuteStep={handleExecuteStep}
        currentStepIndex={currentStepIndex}
        stepStatuses={stepStatuses}
        totalCost={BigInt(preparedPurchase.cost.total.native.value.toString())}
      />
    )}
```

Putting it all together

```typescript
'use client'

import { useState } from 'react'
import { useAccount, useConfig } from 'wagmi'
import { createClient, BlindMintProduct, PreparedPurchase, createAccountViem, createPublicProviderWagmi } from '@manifoldxyz/client-sdk'
import { useWalletClient } from 'wagmi'
import StepModal from './StepModal'

interface MintButtonButtonProps {
  instanceId: string
  quantity?: number
}

export default function MintButton({ instanceId, quantity = 1 }: MintButtonButtonProps) {
  const { address, isConnected } = useAccount()
  const { data: walletClient } = useWalletClient()
  const config = useConfig()
  
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [product, setProduct] = useState<BlindMintProduct | null>(null)
  const [preparedPurchase, setPreparedPurchase] = useState<PreparedPurchase | null>(null)
  const [isModalOpen, setIsModalOpen] = useState(false)
  const [currentStepIndex, setCurrentStepIndex] = useState(0)
  const [stepStatuses, setStepStatuses] = useState<string[]>([])
  const [purchaseComplete, setPurchaseComplete] = useState(false)

  const handlePreparePurchase = async () => {
    if (!address || !walletClient) {
      setError('Please connect your wallet first')
      return
    }

    setIsLoading(true)
    setError(null)
    setPurchaseComplete(false)

    try {
      // Create client with Wagmi public provider
      const publicProvider = createPublicProviderWagmi({ config });
      const client = createClient({ publicProvider });

      const fetchedProduct = await client.getProduct(instanceId) as BlindMintProduct
      setProduct(fetchedProduct)

      const status = await fetchedProduct.getStatus()
      if (status !== 'active') {
        throw new Error(`Product is ${status}`)
      }

      const allocations = await fetchedProduct.getAllocations({ recipientAddress: address })
      if (!allocations.isEligible) {
        throw new Error(allocations.reason || 'Not eligible to mint')
      }

      const prepared = await fetchedProduct.preparePurchase({
        address,
        payload: { quantity }
      })

      setPreparedPurchase(prepared)
      setStepStatuses(new Array(prepared.steps.length).fill('idle'))
      setCurrentStepIndex(0)
      setIsModalOpen(true)
    } catch (err) {
      console.error('Error preparing purchase:', err)
      setError(err instanceof Error ? err.message : 'Failed to prepare purchase')
    } finally {
      setIsLoading(false)
    }
  }

  const handleExecuteStep = async (stepIndex: number) => {
    if (!walletClient || !preparedPurchase || !product) {
      setError('Missing required data')
      return
    }

    const step = preparedPurchase.steps[stepIndex]
    if (!step) {
      setError('Invalid step index')
      return
    }

    setStepStatuses(prev => {
      const newStatuses = [...prev]
      newStatuses[stepIndex] = 'executing'
      return newStatuses
    })

    try {
      const account = createAccountViem({
        walletClient: walletClient as any
      })

      const result = await step.execute(account)

      setStepStatuses(prev => {
        const newStatuses = [...prev]
        newStatuses[stepIndex] = 'completed'
        return newStatuses
      })

      if (stepIndex < preparedPurchase.steps.length - 1) {
        setCurrentStepIndex(stepIndex + 1)
      } else {
        setPurchaseComplete(true)
        setTimeout(() => {
          setIsModalOpen(false)
          setCurrentStepIndex(0)
          setStepStatuses([])
        }, 2000)
      }

      console.log(`Step ${stepIndex + 1} completed:`, result)
    } catch (err) {
      console.error(`Error executing step ${stepIndex + 1}:`, err)
      setStepStatuses(prev => {
        const newStatuses = [...prev]
        newStatuses[stepIndex] = 'failed'
        return newStatuses
      })
      setError(err instanceof Error ? err.message : 'Transaction failed')
    }
  }

  const handleCloseModal = () => {
    if (!stepStatuses.some(status => status === 'executing')) {
      setIsModalOpen(false)
      setPreparedPurchase(null)
      setCurrentStepIndex(0)
      setStepStatuses([])
    }
  }

  return (
    <>
      <div className="flex flex-col items-center gap-4">
        {!isConnected ? (
          <p className="text-gray-600">Please connect your wallet to mint</p>
        ) : (
          <button
            onClick={handlePreparePurchase}
            disabled={isLoading}
            className="px-6 py-3 bg-blue-600 text-white rounded-lg font-semibold hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors"
          >
            {isLoading ? 'Preparing...' : `Mint ${quantity} Blind Box${quantity > 1 ? 'es' : ''}`}
          </button>
        )}

        {error && (
          <div className="p-4 bg-red-50 border border-red-200 rounded-lg">
            <p className="text-red-800 text-sm">{error}</p>
          </div>
        )}

        {purchaseComplete && (
          <div className="p-4 bg-green-50 border border-green-200 rounded-lg">
            <p className="text-green-800 text-sm font-semibold">Purchase completed successfully!</p>
          </div>
        )}
      </div>

      {preparedPurchase && (
        <StepModal
          isOpen={isModalOpen}
          onClose={handleCloseModal}
          steps={preparedPurchase.steps}
          onExecuteStep={handleExecuteStep}
          currentStepIndex={currentStepIndex}
          stepStatuses={stepStatuses}
          formattedCost={preparedPurchase.cost.total.native.formatted}
        />
      )}
    </>
  )
}
```

Best practices:

* Validate the product type using [isBlindMintProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/blind-mint/isblindmintproduct) or [isEditionProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/iseditionproduct) to ensure proper TypeScript typings.
* Run [getStatus](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getstatus) before attempting a purchase to verify the product is available.
* Handle [ClientSDKError](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror) codes for scenarios like ineligibility, sold-out items, or insufficient funds.
* See each method’s documentation for detailed error descriptions.


# Creating a minting bot

The SDK can be used on the server side, enabling use cases such as running a [minting bot](https://help.manifold.xyz/en/articles/11509060-bankrbot)

## Example Scripts

Two ready-to-run bots live in the repository:

* **Edition**: [examples/edition/minting-bot](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/minting-bot)
* **Blind Mint**: [examples/blindmint/minting-bot](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/blindmint/minting-bot)

Each script demonstrates the most direct path to minting—`preparePurchase` followed by `product.purchase()`—so you don’t have to orchestrate transaction steps manually.

### Running an example

1. From the repository root:

   ```bash
   pnpm install
   pnpm build
   ```
2. Inside the example directory:

   ```bash
   pnpm install
   cp .env.example .env
   pnpm start
   ```
3. Fill in the environment variables before running.

## Basic Example

```ts
import {
  createClient,
  createAccountEthers5,
  createPublicProviderEthers5,
  isBlindMintProduct,
  isEditionProduct,
} from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

// Setup provider for the network
const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL!);
const networkId = Number(process.env.NETWORK_ID!); // e.g., 1 for mainnet, 8453 for Base

// Create public provider for the client
const publicProvider = createPublicProviderEthers5({
  [networkId]: provider
});

// Initialize the client
const client = createClient({ publicProvider });

const product = await client.getProduct('INSTANCE_ID');

// Check product status first
const productStatus = await product.getStatus();
if (productStatus !== 'active') {
  throw new Error(`Product is ${productStatus}`);
}

// Handle different product types
if (!isEditionProduct(product)) {
   throw new Error('Unsupported product type');
}

// Setup wallet for signing transactions
const wallet = new ethers.Wallet(process.env.WALLET_PRIVATE_KEY!, provider);
const account = createAccountEthers5({ wallet });

try {
  const prepared = await product.preparePurchase({
    recipientAddress: wallet.address,
    payload: { quantity: 1 },
  });

  const order = await product.purchase({
    account,
    preparedPurchase: prepared,
  });
  console.log(
    order.status,
    order.receipts.map((r) => r.txHash),
  );
} catch (error) {
  console.log(`Unable to execute transaction: ${(error as Error).message}`);
}
```

## Best Practices

* **Type Validation**: Use [isBlindMintProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/blind-mint/isblindmintproduct) or [isEditionProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/iseditionproduct) for proper TypeScript typings
* **Status Checks**: Always run [getStatus](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getstatus) before attempting purchases
* **Error Handling**: Properly handle [ClientSDKError](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror) codes
* **Gas Management**: Monitor gas prices and set appropriate limits
* **Retry Logic**: Implement retry mechanisms for transient failures
* **Security**: Never commit private keys, use environment variables

## Environment Configuration

```env
# Required
WALLET_PRIVATE_KEY=your_private_key_here
INSTANCE_ID=your_product_instance_id
NETWORK_ID=8453
RPC_URL=https://base-mainnet.g.alchemy.com/v2/YOUR_KEY

# RPC Endpoints

# Optional
MINT_QUANTITY=1
```

## Resources

* [**Edition Minting Bot**](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/minting-bot) - Minimal Edition minting script
* [**Blind Minting Bot**](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/blindmint/minting-bot) - Minimal Blind Mint minting script
* [**Examples Overview**](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples) - Directory of all SDK examples
* See method documentation for detailed error descriptions


# FAQ

<details>

<summary><strong>Which networks are supported?</strong> </summary>

Ethereum (1), Base (8453), Optimism (10), Shape (360), Sepolia testnet.

</details>

<details>

<summary><strong>How do I check allowlist eligibility?</strong> </summary>

Call `product.`[`getAllocations`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getallocations)`({ recipientAddress })`.

</details>

<details>

<summary><strong>What happens if a step fails?</strong> </summary>

Catch the error, show context to users, and allow retries. Transaction steps expose [`execute`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute)`()`.

</details>

<details>

<summary><strong>Where do I file issues?</strong> </summary>

[GitHub Issues](https://github.com/manifoldxyz/client-sdk) or Manifold support.

</details>


# \[for AI agents and LLMS] Checklist and guidelines

### Purpose

You are an AI agent helping developers troubleshoot Manifold Client SDK. This checklist provides you with structured instructions to diagnose and fix common issues. Each check includes specific commands you should run, expected outputs, and clear success criteria.

### General Guidelines

As an AI agent, follow these guidelines here:

* **Ask clarifying questions** until you have high confidence in the task. Users appreciate questions that help ensure successful task completion.
* **Be specific** when something is unclear or inaccessible. Ask for file paths, URLs, or specific error messages.
* **Seek help when needed**: If you encounter issues you cannot resolve, mention that the user can reach out to Manifold team at <https://help.manifold.xyz/en/> for additional support.
* **Verify assumptions** before making changes. It's better to confirm than to proceed with uncertainty.

### Common LLM Pitfalls

When helping developers with Manifold Client SDK:

* When executing onchain transactions, use the [`execute`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute) function from each step returned by [`preparePurchase`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/blind-mint/preparepurchase), or use the [`purchase`](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/purchase) function from the [product](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product).
* **ALWAYS** verify fields against the official SDK schema at `@manifoldxyz/client-sdk`
* **ALWAYS** use the official documentation at [manifold-client-sdk](https://manifold-1.gitbook.io/manifold-client-sdk)

If using example code, ensure it's from:

* Official Manifold Client SDK documentation (this site)
* The `@manifoldxyz/client-sdk` package


# Release Notes

### November 18, 2025 (0.5.0-beta.0)

* **New**: Added `subscribeToContractEvents` method to `IPublicProvider` for subscribing to contract events
* **New**: Implemented `subscribeToContractEvents` in all provider adapters (Viem, Ethers5, Wagmi)
* **Feature**: Support for real-time event monitoring with topic filtering and callback handlers

### November 5, 2025 (0.3.1-beta.0)

* **Breaking Change**: Removed `httpRPCs` dependency from SDK client initialization
* **New**: Added public provider abstraction for blockchain interactions
* **New**: Added `createPublicProviderViem` and `createPublicProviderEthers5` functions
* **New**: Added `createPublicProviderWagmi` function for Wagmi integration
* **New**: Added fallback provider support for automatic failover on provider errors or network mismatches
* **Change**: `createClient` now requires a `publicProvider` parameter
* **Change**: Account adapters no longer require the client instance
* **Improvement**: Cleaner separation between read-only and transaction operations
* **Improvement**: Better multi-network support through provider abstraction
* **Improvement**: Enhanced reliability with automatic fallback to backup providers
* **Improvement**: Native Wagmi support for seamless React integration

### October 31, 2025 (0.2.1-beta.4)

* Added support for [Edition product](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product)

### October 21, 2025 (0.1.0-beta.1)

* Added support for [BlindMint product](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/sdk/product/product-types/blind-mint)


# Manifold Client

### Client Creation

**createClient(config)** → ManifoldClient

Creates a new SDK client instance.

#### Parameters

| Parameter         | Type            | Required | Description                           |
| ----------------- | --------------- | -------- | ------------------------------------- |
| config            | object          | ✅        | Configuration object                  |
| └─ publicProvider | IPublicProvider | ✅        | Provider for blockchain interactions  |
| └─ debug          | boolean         | ❌        | Enable debug logging (default: false) |

#### Returns: ManifoldClient

| Property                                                                                        | Type     | Description   |
| ----------------------------------------------------------------------------------------------- | -------- | ------------- |
| [getProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client/getproduct) | function | Get a product |

#### Example with Wagmi

```typescript
import { createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { mainnet, base } from '@wagmi/core/chains';

// Create Wagmi config with multiple chains
const config = createConfig({
  chains: [mainnet, base],
  transports: {
    [mainnet.id]: http('YOUR_MAINNET_RPC_URL'),
    [base.id]: http('YOUR_BASE_RPC_URL'),
  },
});

// Create the public provider
const publicProvider = createPublicProviderWagmi({ config });

// Initialize the Manifold client
const client = createClient({ publicProvider });
```

#### Example with Viem

```typescript
import { createClient, createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';

// Create public clients for each network you want to support
const publicClient = createPublicClient({
  chain: mainnet,
  transport: http('YOUR_RPC_URL')
});

// Create the public provider
const publicProvider = createPublicProviderViem({ 
  1: publicClient // mainnet
});

// Initialize the Manifold client
const client = createClient({ publicProvider });
```

#### Example with Ethers v5

```typescript
import { createClient, createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

// Create ethers providers for each network
const provider = new ethers.providers.JsonRpcProvider('YOUR_RPC_URL');

// Create the public provider
const publicProvider = createPublicProviderEthers5({ 
  1: provider // mainnet
});

// Initialize the Manifold client
const client = createClient({ publicProvider });
```


# getProduct

**getProduct(instanceId | url)** → [Product](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product)

Fetches detailed product information.

#### Parameters

| Parameter         | Type   | Required | Description                          |
| ----------------- | ------ | -------- | ------------------------------------ |
| instanceId \| url | string | ✅        | The instanceId or url of the product |

#### Returns: [Product](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product)

#### Example

<pre class="language-typescript"><code class="lang-typescript">import { isBlindMintProduct, createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { mainnet } from '@wagmi/core/chains';

// Setup Wagmi config
const config = createConfig({
  chains: [mainnet],
  transports: {
    [mainnet.id]: http('YOUR_RPC_URL')
  }
});

// Create public provider
const publicProvider = createPublicProviderWagmi({ config });

// Create client
const client = createClient({ publicProvider });

<strong>const product = await client.getProduct('4150231280');
</strong>console.log(`AppType: ${product.type}`);
</code></pre>

[**ClientSDKError**](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror)

| Code              | Message                  |
| ----------------- | ------------------------ |
| NOT\_FOUND        | product not found        |
| UNSUPPORTED\_TYPE | Unsupported product type |


# Product

Calling [getProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client/getproduct) returns a product object with a consistent structure across all Manifold app types.

| Field       | Type                                                                                      | Required | Description                        |
| ----------- | ----------------------------------------------------------------------------------------- | -------- | ---------------------------------- |
| **id**      | string                                                                                    | ✅        | Instance ID                        |
| type        | [AppType](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/apptype)           | ✅        | Type of the product                |
| data        | [InstanceData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/instancedata) | ✅        | Product offchain data              |
| previewData | [PreviewData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/previewdata)   | ✅        | Return preview data of the product |

Product instances are created based on their specific type (**Edition** or **Blind Mint**). Each specialization adds additional methods and type guards while preserving the shared core API.


# Common

The following methods apply to both [Edition](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/sdk/product/product-types/edition-product) and [Blind Mint](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/sdk/product/product-types/blind-mint) product

* purchase
* getStatus
* getAllocations
* getInventory
* getRules
* getProvenance


# purchase

purchase

**purchase(params)** → [Receipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/receipt)

Initiates a purchase for the specified product.

This method may trigger multiple write transactions (e.g., token approval and minting).

#### Parameters

| Parameter        | Type                                                                                              | Required | Description                                                                                                                                                      |
| ---------------- | ------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| account          | [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account)                   | ✅        | Buyer’s account                                                                                                                                                  |
| preparedPurchase | [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase) | ✅        | Prepared transaction object returned from [preparePurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/preparepurchase) call |
| confirmations    | number                                                                                            | ❌        | Number of confirmation blocks (Default 1)                                                                                                                        |

#### Returns: [Receipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/receipt)

The receipt contains the confirmed transaction metadata under `transactionReceipt` and an `order` object with parsed token details for the mint that was executed.

#### Example

```tsx
const receipt = await product.purchase({
  account,
  preparedPurchase,
});

console.log('Mint transaction:', receipt.transactionReceipt.txHash);

if (receipt.order) {
  console.log('Recipient:', receipt.order.recipientAddress);
  for (const item of receipt.order.items) {
    console.log(`Minted token ${item.token.tokenId} x${item.quantity}`);
  }
}
```

[**Errors**](https://www.notion.so/Manifold-Client-SDK-Complete-Developer-Guide-2676b055ee58800abc38ccd30cdfca70?pvs=21)

<table><thead><tr><th width="234.6640625">Code</th><th width="175.30859375">Message</th><th width="142.54296875">data</th><th>metadata</th></tr></thead><tbody><tr><td>TRANSACTION_FAILED</td><td>transaction failed</td><td><a href="https://docs.ethers.org/v5/api/utils/logger/#errors--call-exception">CallExceptions</a></td><td>{ receipts : <a href="../../../reference/receipt">Receipt</a>[]} (For completed steps)</td></tr><tr><td>TRANSACTION_REVERTED</td><td>transaction reverted</td><td><a href="https://docs.ethers.org/v5/api/utils/logger/#errors--call-exception">CallExceptions</a></td><td>{ receipts : <a href="../../../reference/receipt">Receipt</a>[]} (For completed steps)</td></tr><tr><td>TRANSACTION_REJECTED</td><td>user rejected transaction</td><td></td><td>{ receipts : <a href="../../../reference/receipt">Receipt</a>[]} (For completed steps)</td></tr><tr><td>INSUFFIENT_FUNDS</td><td>wallet does not have sufficient funds for purchase</td><td></td><td>{ receipts : <a href="../../../reference/receipt">Receipt</a>[]} (For completed steps)</td></tr><tr><td>LEDGER_ERROR</td><td>error with ledger wallet, make sure blind signing is on</td><td></td><td>{ receipts : <a href="../../../reference/receipt">Receipt</a>[]} (For completed steps)</td></tr></tbody></table>


# getStatus

**getStatus()** → StatusResponse

Retrieves the current status of the product.

#### Returns:&#x20;

```typescript
'active' | 'upcoming' | 'sold-out' | 'ended'
```

* **active**: The product is currently active and available for purchase.
* **upcoming**: The product sale has not started yet.
* **sold-out**: The product is sold out.
* **ended**: The product sale has ended.

#### Example

```jsx
const status = await product.getStatus();
console.log(`Current product status ${status}`)
```


# getAllocations

**getAllocations(params)** → AllocationResponse

Retrieves the allocation quantity for a given wallet address.

#### Parameters

| Parameter            | Type   | Required | Description    |
| -------------------- | ------ | -------- | -------------- |
| **recipientAddress** | string | ✅        | Buyer’s wallet |

#### Returns: AllocationResponse

| Field      | Type    | Required | Description                                                      |
| ---------- | ------- | -------- | ---------------------------------------------------------------- |
| isEligible | boolean | ✅        | Can purchase?                                                    |
| reason     | string  | ❌        | Why not eligible                                                 |
| quantity   | number  | ✅        | Quantity eligible. A value of `-1` indicates unlimited quantity. |

#### Example

```jsx
const allocations = await product.getAllocations**({
  recipientAddress: '0x742d35Cc...'
  });
if (!allocations.isEligible) {
  console.log('Cannot mint:', allocations.reason);  
  return;
}
console.log('Total alloted:', allocations.quantity);
```

[**Errors**](https://www.notion.so/Manifold-Client-SDK-Complete-Developer-Guide-2676b055ee58800abc38ccd30cdfca70?pvs=21)

| Code           | Message                     |
| -------------- | --------------------------- |
| INVALID\_INPUT | `invalid recipient address` |

*


# getInventory

**getInventory()** → [ProductInventory](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productinventory)

Retrieves the product’s total supply and total number of purchases.

#### Returns: [ProductInventory](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productinventory)

| Field          | Type   | Required | Description                                                        |
| -------------- | ------ | -------- | ------------------------------------------------------------------ |
| totalSupply    | number | ✅        | Total product supply.  A value of `-1` indicates unlimited supply. |
| totalPurchased | number | ✅        | Total product purchased                                            |


# getRules

**getRules()** → ProductRule

Retrieves the product rules, such as start and end dates, maximum tokens per wallet, audience restrictions, and more.

#### Returns: ProductRule

| Field               | Type   | Required | Description                                        |
| ------------------- | ------ | -------- | -------------------------------------------------- |
| startDate           | Date   | ❌        | Start date (if not provided, start immediately)    |
| endDate             | Date   | ❌        | End date (if not provided, the product never ends) |
| audienceRestriction | enum   | ✅        | allowlist \| none                                  |
| maxPerWallet        | number | ❌        | Number of allowed purchased per wallet             |

**AudienceRestriction**

* `allowlist`: The product is restricted to specific wallet addresses.
* `none`: The product has no audience restrictions.


# getProvenance

**getProvenance()** → [ProductProvenance](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productprovenance)

Retrieves provenance information for the product, such as the related contract address, token ID, creator details, and more.

#### Returns: [ProductProvenance](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productprovenance)

| Field     | Type                                                                              | Required | Description                                  |
| --------- | --------------------------------------------------------------------------------- | -------- | -------------------------------------------- |
| creator   | [Creator](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/creator)   | ✅        | Information about the creator of the product |
| contract  | [Contract](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/contract) | ❌        | Information about the contract the product   |
| token     | [Token](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/token)       | ❌        | Information about the token of the product   |
| networkId | number                                                                            | ❌        | Network ID of the product                    |


# Edition Product

Follow [this guide](https://help.manifold.xyz/en/articles/9387344-create-an-edition-open-or-limited) to create an Edition product.

This data is returned when calling the [getProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client/getproduct) method.

#### Handling different configurations

An Edition product can be created or updated with various configurations:

* **Price:** Can be set in ETH or any ERC-20 token.
* **Total Supply:** Can be unlimited or limited.
* **Supply per Wallet:** The number of tokens a single wallet can mint.
* **Start/End Date:** Defines the timeline for the drop.
* **Audience:**
  * **anyone:** Anyone can purchase.
  * **allowlist:** Only a predefined list of wallet addresses can purchase.

The SDK provides convenient methods to handle these configurations:

[preparePurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/preparepurchase)

Performs all necessary checks to ensure the purchase is valid and throws appropriate errors if any validation fails.

**Examples of Thrown Errors:**

* `ErrorCode.NOT_STARTED` — The product start date is in the future.
* `ErrorCode.ENDED` — The product end date has passed.
* `ErrorCode.SOLD_OUT` — The product is sold out (based on total supply).
* `ErrorCode.NOT_ELIGIBLE` — The recipient is not on the allowlist.
* `ErrorCode.INVALID_INPUT` — The desired purchase quantity exceeds the per-wallet limit or total supply.
* `ErrorCode.INSUFFICIENT_FUNDS` — The account does not have enough ETH or ERC-20 tokens for the purchase.

[getStatus](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getstatus)

Useful for displaying the current product status (e.g., `active`, `upcoming`, `sold-out`, `ended`).

[getAllocations](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getallocations)

Useful for showing the total quantity an [account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) is eligible to purchase.


# preparePurchase

**preparePurchase(params)** → [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase)

Simulates a purchase to check eligibility and calculate the total cost.

#### Parameters

<table><thead><tr><th width="182.84375">Parameter</th><th width="181.56640625">Type</th><th width="97.63671875">Required</th><th>Description</th></tr></thead><tbody><tr><td>userAddress</td><td>string</td><td>✅</td><td>The address making the purchase</td></tr><tr><td>recipientAddress</td><td>string</td><td>❌</td><td>If different than <code>address</code></td></tr><tr><td>networkId</td><td>number</td><td>❌</td><td>If specify, forced transaction on the network (handle funds bridging automatically), assume product network otherwise</td></tr><tr><td>payload</td><td>{quantity: number}</td><td>✅</td><td>Specific to Edition Products. Specify quantity of purchase</td></tr><tr><td><strong>gasBuffer</strong></td><td>object</td><td>❌</td><td>How much additional gas to spend on the purchase</td></tr><tr><td>gasBuffer.fixed</td><td>BigInt</td><td>❌</td><td>Fixed gas buffer amount</td></tr><tr><td>gasBuffer.multiplier</td><td>number</td><td>❌</td><td><p>Gas buffer by multiplier. </p><p>The multiplier represents a percentage (as a number out of 100). For example:</p><ul><li>multiplier: 120 means 120% of the original estimate (20% increase)</li><li>multiplier: 150 means 150% of the original estimate (50% increase)</li></ul></td></tr><tr><td>account</td><td><a href="https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/references/account">Account</a></td><td>❌</td><td>If provided, it will perform balance checks on the specified account; otherwise, it will skip balance checks.</td></tr></tbody></table>

#### Returns: [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase)

> A purchase can involve more than one transaction.\
> For example, minting and paying with ERC-20 tokens requires an approval transaction followed by a mint transaction.\
> If you’re building your own front end with the SDK, you may want users to trigger these transactions explicitly (e.g., by clicking separate buttons).
>
> [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase) returns a list of steps for this purpose.\
> Each step represents an on-chain transaction that can be executed by calling [step.execute](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute)().\
> Each `execute` call performs the necessary on-chain checks to determine whether the transaction is still required; if it isn’t, the step is skipped. [Click here to learn more](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps)

#### Example

<pre class="language-jsx"><code class="lang-jsx">import { createClient, createPublicProviderViem, type AppType } from '@manifoldxyz/client-sdk'
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const publicClient = createPublicClient({
  chain: mainnet,
  transport: http(),
})
const publicProvider = createPublicProviderViem({ 1: publicClient })
const client = createClient({ publicProvider });

const product = await client.getProduct('12311232')
if (product.type !== AppType.Edition) {
	throw new Error(`Unsupported app type`)
}
try {
<strong>  const preparedPurchase = await product.preparePurchase&#x3C;EditionPayload>({
</strong>    userAddress: '0x....', // the connected wallet
    payload: {
	  quantity: 1
    },
    gasBuffer: {
     multiplier: 0.25 // 25% gas buffer
    }
<strong>  });
</strong>} catch (error: ClientSDKError) {
  console.log(`Error: ${error.message}`)
  return
}

console.log('Total cost:', preparedPurchase.cost.total.formatted);
</code></pre>

[**Errors**](https://www.notion.so/Manifold-Client-SDK-Complete-Developer-Guide-2676b055ee58800abc38ccd30cdfca70?pvs=21)

<table><thead><tr><th>Code</th><th width="338.7421875">Message</th><th>data</th></tr></thead><tbody><tr><td>INVALID_INPUT</td><td><code>invalid input</code></td><td></td></tr><tr><td>UNSUPPORTED_NETWORK</td><td><code>unsupported networkId ${networkId}</code></td><td></td></tr><tr><td>NOT_ELIGIBLE</td><td><code>wallet not eligible to purchase product</code></td><td>Eligibility</td></tr><tr><td>SOLD_OUT</td><td><code>product sold out</code></td><td></td></tr><tr><td>LIMIT_REACHED</td><td><code>you've reached your purchase limit</code></td><td></td></tr><tr><td>ENDED</td><td><code>ended</code></td><td></td></tr><tr><td>NOT_STARTED</td><td><code>not started, come back later</code></td><td></td></tr><tr><td>ESTIMATION_FAILED</td><td><code>transaction estimation failed</code></td><td><a href="https://docs.ethers.org/v5/api/utils/logger/#errors--call-exception">CallExceptions</a></td></tr></tbody></table>

Eligibiliy

| Field        | Type                                                                           | Description                                                                 |
| ------------ | ------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| reason       | string                                                                         | Why not eligible                                                            |
| missingFunds | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money)\[] | Missing required funds to purchase (Could be in native ETH or ERC20 tokens) |


# fetchOnchainData

**fetchOnchainData()** → [EditionOnchainData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editiononchaindata)

Get on-chain data and assign `onchainData` properties for the product object.

#### Example

```tsx
const product = await sdk.getProduct('31231232')
await product.fetchOnchainData()
console.log(`cost: ${product.onchainData.cost.formatted}`)
```


# isEditionProduct

isEditionProduc&#x74;**()** → boolean

Validate whether a [product](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product) is an [Edition product](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editionproduct)

Provides additional TypeScript typing support by narrowing the product type to [EditionProduct](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/references/editionproduct)

#### Example

```tsx
const product = await sdk.getProduct('31231232')
if (!isEditionProduct(product)) {
  throw new Error('Is not an edition instance')
}
// product is now EditionProduct
```


# Blind Mint

Blind Mints offer randomized, gacha-style minting.

Follow [this guide](https://help.manifold.xyz/en/articles/9449681-serendipity) to create a Blind Mint product.

This data is returned when calling the [getProduct](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/manifold-client/getproduct) method

#### Handling different configurations

A Blind Mint product can be created or updated with various configurations:

* **Price:** Can be set in ETH.
* **Total Supply:** Can be unlimited or limited.
* **Start/End Date:** Defines the timeline for the drop.
* **Audience:**
  * **anyone:** Anyone can purchase.

The SDK provides convenient methods to handle these configurations:

[preparePurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/preparepurchase)

Performs all necessary checks to ensure the purchase is valid and throws appropriate errors if any validation fails.

**Examples of Thrown Errors:**

* `ErrorCode.NOT_STARTED` — The product start date is in the future.
* `ErrorCode.ENDED` — The product end date has passed.
* `ErrorCode.SOLD_OUT` — The product is sold out (based on total supply).
* `ErrorCode.INSUFFICIENT_FUNDS` — The account does not have enough ETH for the purchase.

[getStatus](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getstatus)

Useful for displaying the current product status (e.g., `active`, `upcoming`, `sold-out`, `ended`).

[getAllocations](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getallocations)

Useful for showing the total quantity an [account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) is eligible to purchase.


# preparePurchase

**preparePurchase(params)** → [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase)

Simulates purchase to check eligibility and get total cost.

#### Parameters

<table><thead><tr><th width="181.0078125">Parameter</th><th width="168.01171875">Type</th><th width="107.44140625">Required</th><th>Description</th></tr></thead><tbody><tr><td>userAddress</td><td>string</td><td>✅</td><td>The address making the purchase</td></tr><tr><td>recipientAddress</td><td>string</td><td>❌</td><td>If different than <code>address</code></td></tr><tr><td>networkId</td><td>number</td><td>❌</td><td>If specify, forced transaction on the network (handle funds bridging automatically), assume product network otherwise</td></tr><tr><td>payload</td><td>{quantity: number}</td><td>✅</td><td>Specific to Blind Mint Products. Specify quantity of purchase</td></tr><tr><td><strong>gasBuffer</strong></td><td>object</td><td>❌</td><td>How much additional gas to spend on the purchase</td></tr><tr><td>gasBuffer.fixed</td><td>BigInt</td><td>❌</td><td>Fixed gas buffer amount</td></tr><tr><td>gasBuffer.multiplier</td><td>number</td><td>❌</td><td><p></p><p>Gas buffer by multiplier. </p><p>The multiplier represents a percentage (as a number out of 100). For example:</p><ul><li>multiplier: 120 means 120% of the original estimate (20% increase)</li><li>multiplier: 150 means 150% of the original estimate (50% increase)</li></ul></td></tr><tr><td>account</td><td><a href="../../../reference/account">Account</a></td><td>❌</td><td>If provided, it will perform balance checks on the specified account; otherwise, it will skip balance checks.</td></tr></tbody></table>

#### Returns: [PreparedPurchase](https://www.notion.so/Manifold-Client-SDK-Complete-Developer-Guide-2676b055ee58800abc38ccd30cdfca70?pvs=21)

#### Example

```jsx
import { createClient, createPublicProviderViem, isBlindMintProduct } from '@manifoldxyz/client-sdk'
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const publicClient = createPublicClient({
  chain: mainnet,
  transport: http(),
})
const publicProvider = createPublicProviderViem({ 1: publicClient })
const client = createClient({ publicProvider });

const product = await client.getProduct('12311232')
if (!isBlindMintProduct(product)) {
  throw new Error(`Unsupported app type`)
}
try {
  const preparedPurchase = await product.preparePurchase({
    userAddress: '0x....', // the connected wallet
    payload: {
      quantity: 1
    },
    gasBuffer: {
     multiplier: 0.25 // 25% gas buffer
    }
  });
} catch (error: ClientSDKError) {
  console.log(`Error: ${error.message}`)
  return
}

console.log('Total cost:', preparedPurchase.cost.total.formatted);
```

[**Errors**](https://www.notion.so/Manifold-Client-SDK-Complete-Developer-Guide-2676b055ee58800abc38ccd30cdfca70?pvs=21)

| Code                 | Message                              | data                                                                                  |
| -------------------- | ------------------------------------ | ------------------------------------------------------------------------------------- |
| INVALID\_INPUT       | `invalid input`                      |                                                                                       |
| UNSUPPORTED\_NETWORK | `unsupported networkId ${networkId}` |                                                                                       |
| SOLD\_OUT            | `product sold out`                   |                                                                                       |
| LIMIT\_REACHED       | `you've reached your purchase limit` |                                                                                       |
| ENDED                | `ended`                              |                                                                                       |
| NOT\_STARTED         | `not started, come back later`       |                                                                                       |
| ESTIMATION\_FAILED   | `transaction estimation failed`      | [CallExceptions](https://docs.ethers.org/v5/api/utils/logger/#errors--call-exception) |


# fetchOnchainData

**fetchOnchainData()** → [BlindMintOnchainData](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/references/blindmintonchaindata)

Retrieves on-chain data and assigns the `onchainData` properties to the [product](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product) object.

#### Example

```tsx
const product = await sdk.getProduct('31231232')
await product.fetchOnchainData()
console.log(`cost: ${product.onchainData.cost.formatted}`)
```


# isBlindMintProduct

isBlindMintProduc&#x74;**()** → boolean

Validate whether a [product](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/sdk/product) is a [Blind Mint product](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/sdk/product/product-types/blind-mint)

Provides additional TypeScript typing support by narrowing the product type to [BlindMintProduct](https://app.gitbook.com/o/FkM3zqPi1O0VypWXgiUZ/s/wX9Yl8DLygpenDBVWGPF/~/changes/1/references/blindmintproduct)

#### Example

```tsx
const product = await sdk.getProduct('31231232')
if (!isBlindMintProduct(product)) {
  throw new Error('Is not a blind mint instance')
}
// product is now BlindMintProduct
```


# Public Provider Adapters

Public providers enable the SDK to perform read-only blockchain operations such as fetching balances, estimating gas, reading smart contract data, and subscribing to contract events. They are required when initializing the Manifold Client.

## Available Adapters

* [Wagmi Public Provider](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/public-provider-adapters/wagmi)
* [Viem Public Provider](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/public-provider-adapters/viem)
* [Ethers v5 Public Provider](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/public-provider-adapters/ethersv5)

## Multi-Network Support

The SDK supports multiple networks simultaneously. Provide a public client/provider for each network you want to support:

```typescript
const publicProvider = createPublicProviderViem({
  1: mainnetClient,       // Ethereum Mainnet
  8453: baseClient,       // Base
  10: optimismClient,     // Optimism
  360: shapeClient,       // Shape
  11155111: sepoliaClient // Sepolia Testnet
});
```

## Network Requirements

The public provider must include a client/provider for the network where the NFT product is deployed. The SDK will automatically use the appropriate provider based on the product's network.

[**ClientSDKError**](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror)

| Code                    | Message                                |
| ----------------------- | -------------------------------------- |
| INVALID\_INPUT          | Public provider is required            |
| NETWORK\_NOT\_SUPPORTED | No provider available for network      |
| WRONG\_NETWORK          | Provider is connected to wrong network |


# Wagmi

**createPublicProviderWagmi(params)** → IPublicProvider

Creates a public provider from a Wagmi `Config`. This adapter lets the SDK reuse the public clients that Wagmi configures for read-only blockchain operations such as fetching balances, estimating gas, and reading contracts.

## Parameters

| Parameter | Type                                             | Required | Description                             |
| --------- | ------------------------------------------------ | -------- | --------------------------------------- |
| config    | [Config](https://wagmi.sh/core/api/createConfig) | ✅        | Wagmi config with chains and transports |

## Examples

### Basic usage

```typescript
import { createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { mainnet, base } from '@wagmi/core/chains';

// Configure Wagmi with the networks you intend to support
const config = createConfig({
  chains: [mainnet, base],
  transports: {
    [mainnet.id]: http('YOUR_MAINNET_RPC_URL'),
    [base.id]: http('YOUR_BASE_RPC_URL'),
  },
});

// Create the Manifold public provider
const publicProvider = createPublicProviderWagmi({ config });

// Pass into the Manifold client
const client = createClient({ publicProvider });
```

### Multi-network with fallback transports

```typescript
import { createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig } from '@wagmi/core';
import { fallback, http } from 'viem';
import { mainnet, optimism } from '@wagmi/core/chains';

const config = createConfig({
  chains: [mainnet, optimism],
  transports: {
    [mainnet.id]: fallback([
      http('PRIMARY_MAINNET_RPC_URL'),
      http('BACKUP_MAINNET_RPC_URL'),
    ]),
    [optimism.id]: http('YOUR_OPTIMISM_RPC_URL'),
  },
});

const publicProvider = createPublicProviderWagmi({ config });
```

### Browser usage with Wagmi React

```typescript
import { useMemo } from 'react';
import { WagmiProvider, useConfig } from 'wagmi';
import { createConfig, http } from '@wagmi/core';
import { mainnet } from '@wagmi/core/chains';
import { createClient, createPublicProviderWagmi } from '@manifoldxyz/client-sdk';

const config = createConfig({
  chains: [mainnet],
  transports: {
    [mainnet.id]: http(), // Uses the default public RPC
  },
});

export function App() {
  return (
    <WagmiProvider config={config}>
      <SdkInitializer />
    </WagmiProvider>
  );
}

function SdkInitializer() {
  const config = useConfig();
  const publicProvider = createPublicProviderWagmi({ config });

  return /* your UI */;
}
```

## Event Subscription

Subscribe to contract events in real-time using the `subscribeToContractEvents` method:

```typescript
import { createPublicProviderWagmi } from '@manifoldxyz/client-sdk';
import { createConfig, http } from '@wagmi/core';
import { mainnet } from '@wagmi/core/chains';

const config = createConfig({
  chains: [mainnet],
  transports: {
    [mainnet.id]: http('YOUR_MAINNET_RPC_URL'),
  },
});

const publicProvider = createPublicProviderWagmi({ config });

// Subscribe to Transfer events
const unsubscribe = await publicProvider.subscribeToContractEvents({
  contractAddress: '0x...',
  abi: erc20Abi,
  networkId: 1,
  topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'], // Transfer event signature
  callback: (log) => {
    console.log('Transfer event:', log);
  }
});

// Later: unsubscribe from events
unsubscribe();
```

## Notes

* The Wagmi config **must** include a transport for every chain you expect to access. If a chain is missing, calls will throw `ClientSDKError` with `UNSUPPORTED_NETWORK`.
* The adapter uses Wagmi's `getPublicClient` under the hood, so the config should expose a public client transport (e.g., `http`, `fallback`).
* Wagmi handles provider caching; reuse the same config instance when possible to avoid creating redundant clients.


# Viem

**createPublicProviderViem(publicClients, fallbackProviders?)** → IPublicProvider

Creates a public provider from Viem public clients with optional fallback support.

## Parameters

| Parameter     | Type                                             | Required | Description                               |
| ------------- | ------------------------------------------------ | -------- | ----------------------------------------- |
| publicClients | Record\<number, PublicClient \| PublicClient\[]> | ✅        | Map of network IDs to Viem public clients |

## Examples

### Basic usage

```typescript
import { createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet, base, optimism } from 'viem/chains';

// Create public clients for each network
const publicClients = {
  1: createPublicClient({
    chain: mainnet,
    transport: http('YOUR_MAINNET_RPC_URL')
  }),
  8453: createPublicClient({
    chain: base,
    transport: http('YOUR_BASE_RPC_URL')
  }),
  10: createPublicClient({
    chain: optimism,
    transport: http('YOUR_OPTIMISM_RPC_URL')
  })
};

// Create the public provider
const publicProvider = createPublicProviderViem(publicClients);

// Use with Manifold client
const client = createClient({ publicProvider });
```

### With fallback providers

```typescript
import { createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet, base } from 'viem/chains';

// Providers with fallback (used when primary fails or is on wrong network)
const publicClients = {
  1: [
  createPublicClient({
    chain: mainnet,
    transport: http('PRIMARY_MAINNET_RPC_URL')
  }),
  createPublicClient({
    chain: mainnet,
    transport: http('FALLBACK_MAINNET_RPC_URL')
  })
  ]
};

// Create the public provider with fallback support
const publicProvider = createPublicProviderViem(publicClients);

// Use with Manifold client
const client = createClient({ publicProvider });
```

## Browser usage

For browser applications using MetaMask or other injected wallets:

```typescript
import { createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, custom } from 'viem';
import { mainnet } from 'viem/chains';

// Use the browser's injected provider
const publicClient = createPublicClient({
  chain: mainnet,
  transport: custom(window.ethereum)
});

const publicProvider = createPublicProviderViem({
  1: publicClient
});

const client = createClient({ publicProvider });
```

## Event Subscription

Subscribe to contract events in real-time using the `subscribeToContractEvents` method:

```typescript
import { createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';

const publicClient = createPublicClient({
  chain: mainnet,
  transport: http('YOUR_MAINNET_RPC_URL')
});

const publicProvider = createPublicProviderViem({
  1: publicClient
});

// Subscribe to Transfer events
const unsubscribe = await publicProvider.subscribeToContractEvents({
  contractAddress: '0x...',
  abi: erc20Abi,
  networkId: 1,
  topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'], // Transfer event signature
  callback: (log) => {
    console.log('Transfer event:', log);
  }
});

// Later: unsubscribe from events
unsubscribe();
```


# Ethers v5

**createPublicProviderEthers5(providers, fallbackProviders?)** → IPublicProvider

Creates a public provider from Ethers v5 providers with optional fallback support.

## Parameters

| Parameter | Type                                                                                                                                                                                                               | Required | Description                            |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | -------------------------------------- |
| providers | Record\<number, [JsonRPCProvider](https://docs.ethers.org/v5/api/providers/jsonrpc-provider/#JsonRpcProvider) \| [JsonRPCProvider](https://docs.ethers.org/v5/api/providers/jsonrpc-provider/#JsonRpcProvider)\[]> | ✅        | Map of network IDs to Ethers providers |

## Examples

### Basic usage

```typescript
import { createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

// Create providers for each network
const providers = {
  1: new ethers.providers.JsonRpcProvider('YOUR_MAINNET_RPC_URL'),
  8453: new ethers.providers.JsonRpcProvider('YOUR_BASE_RPC_URL'),
  10: new ethers.providers.JsonRpcProvider('YOUR_OPTIMISM_RPC_URL')
};

// Create the public provider
const publicProvider = createPublicProviderEthers5(providers);

// Use with Manifold client
const client = createClient({ publicProvider });
```

### With fallback providers

```typescript
import { createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

// Primary providers
const providers = {
  1: [new ethers.providers.JsonRpcProvider('PRIMARY_MAINNET_RPC_URL'), 
  new ethers.providers.JsonRpcProvider('SECONDARY_MAINNET_RPC_URL')
  ],
};

// Create the public provider with fallback support
const publicProvider = createPublicProviderEthers5(providers);

// Use with Manifold client
const client = createClient({ publicProvider });
```

## Event Subscription

Subscribe to contract events in real-time using the `subscribeToContractEvents` method:

```typescript
import { createPublicProviderEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const provider = new ethers.providers.JsonRpcProvider('YOUR_MAINNET_RPC_URL');

const publicProvider = createPublicProviderEthers5({
  1: provider
});

// Subscribe to Transfer events
const unsubscribe = await publicProvider.subscribeToContractEvents({
  contractAddress: '0x...',
  abi: erc20Abi,
  networkId: 1,
  topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'], // Transfer event signature
  callback: (log) => {
    console.log('Transfer event:', log);
  }
});

// Later: unsubscribe from events
unsubscribe();
```


# Account Adapters

The SDK requires two types of blockchain connections:

## Public Providers

[Public providers](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/public-provider-adapters) enable read-only blockchain operations like fetching balances, estimating gas, and reading contracts. They are **required** when initializing the Manifold Client.

* [Viem Public Provider](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/public-provider-adapters/viem) - For Viem users
* [Ethers v5 Public Provider](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/public-provider-adapters/ethersv5) - For Ethers v5 users

## Account Adapters

An [account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) is required to execute on-chain transactions.\
Both [**purchase**](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/purchase) and [**execute**](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute) operations require an account, as they involve triggering transactions on the user's behalf.\
The SDK provides convenient methods for creating an account using popular Web3 libraries:

* [**Viem**](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/account-adapters/viem) - Modern, TypeScript-first library
* [**Ethers v5**](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/account-adapters/ethersv5) - Popular, battle-tested library

## Quick Start

```typescript
import { createClient, createPublicProviderViem, createAccountViem } from '@manifoldxyz/client-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { mainnet } from 'viem/chains';

// 1. Create public provider for blockchain reads
const publicClient = createPublicClient({
  chain: mainnet,
  transport: http('YOUR_RPC_URL')
});
const publicProvider = createPublicProviderViem({ 1: publicClient });

// 2. Initialize Manifold client
const client = createClient({ publicProvider });

// 3. Create account for transactions (when needed)
const walletClient = createWalletClient({ /* ... */ });
const account = createAccountViem({ walletClient });
```


# EthersV5

**createAccountEthers5(params)** → [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account)

Creates an account representation from an **Ethers v5** signer or wallet.\
(You only need to provide either a signer **or** a wallet.)

#### Parameters

| Parameter | Type                                                                                      | Required | Description                                                                                           |
| --------- | ----------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------- |
| signer    | [JsonRpcSigner](https://docs.ethers.org/v5/api/providers/jsonrpc-provider/#JsonRpcSigner) | ❌        | Instance of [JsonRpcSigner](https://docs.ethers.org/v5/api/providers/jsonrpc-provider/#JsonRpcSigner) |
| wallet    | [Wallet](https://docs.ethers.org/v5/api/signer/#Wallet)                                   | ❌        | Useful for server-side applications using programmatic wallet generation.                             |

#### Returns: [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account)

#### Example

<pre class="language-typescript"><code class="lang-typescript">import { createAccountEthers5, createClient, createPublicProviderEthers5, EditionProduct } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

// Create provider for the network
const provider = new ethers.providers.JsonRpcProvider('YOUR_RPC_URL');

// Create public provider for the client
const publicProvider = createPublicProviderEthers5({
  1: provider // mainnet
});

// Initialize the client
const client = createClient({ publicProvider });

<strong>const product = await client.getProduct('4150231280') as EditionProduct;
</strong>
// Create wallet for signing
const wallet = new ethers.Wallet('wallet-private-key', provider);

const prepared = await product.preparePurchase({
  address: wallet.address,
  payload: {
    quantity: 1
  },
});

// Create account from wallet
const account = createAccountEthers5({
  wallet
})

const order = await product.purchase({
  account,
  preparedPurchase: prepared,
});
</code></pre>

[**ClientSDKError**](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror)

| Code           | Message                                                                   |
| -------------- | ------------------------------------------------------------------------- |
| INVALID\_INPUT | Provide either signer or wallet, not both \| Signer or wallet is required |


# Viem

**createAccountViem(params)** → [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account)

Create an account representation from [Viem Wallet Client](https://viem.sh/docs/clients/wallet)

#### Parameters

| Parameter    | Type                                                | Required | Description                    |
| ------------ | --------------------------------------------------- | -------- | ------------------------------ |
| walletClient | [WalletClient](https://viem.sh/docs/clients/wallet) | ✅        | Instance of Viem Wallet client |

#### Returns: [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account)

#### Example

{% tabs %}
{% tab title="index.ts" %}

```typescript
import { createClient, BlindMintProduct, createAccountViem, createPublicProviderViem } from '@manifoldxyz/client-sdk';
import { walletClient, publicClient } from './client.ts';

// Create public provider for blockchain interactions
const publicProvider = createPublicProviderViem({
  1: publicClient // mainnet
});

// Initialize client with public provider
const client = createClient({ publicProvider });

// Grab product
const product = await client.getProduct('4150231280') as BlindMintProduct;

const prepared = await product.preparePurchase({
  address: '0xBuyer',
  payload: { quantity: 1 },
});

const account = createAccountViem({
  walletClient
})
const order = await product.purchase({
  account,
  preparedPurchase: prepared,
});
const txHash = order.receipts[0]?.txHash;
console.log(`Transaction submitted ${txHash}`)
```

{% endtab %}

{% tab title="client.ts" %}

```typescript
import { createWalletClient, createPublicClient, custom, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

// Create public client for read operations
export const publicClient = createPublicClient({
  chain: mainnet,
  transport: http('YOUR_RPC_URL') // or custom(window.ethereum) for browser
})

// Create wallet client for transactions
const account = privateKeyToAccount('0x...') 
export const walletClient = createWalletClient({
  account, 
  chain: mainnet,
  transport: custom(window.ethereum)
})
```

{% endtab %}
{% endtabs %}

[**ClientSDKError**](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/clientsdkerror)

| Code           | Message                                                                                |
| -------------- | -------------------------------------------------------------------------------------- |
| INVALID\_INPUT | Account not found. Please provide an Account explicitly when creating the Viem client. |


# Transaction Steps

## Introduction

Some product purchases may require more than one transaction to complete. For example, Edition products configured with ERC20 payments require:

1. **ERC20 spend approval**: Grant contract permission to spend user tokens
2. **Mint**: Execute the actual purchase transaction

Other scenarios include cross-chain purchases (bridge, then mint) or batch operations.

## Execution Methods

### 1. Built-in Execution (Recommended)

For **user-facing apps**, the SDK separates transactions into explicit steps. This gives you:

* Progress tracking for multi-step flows
* Fine-grained error handling per transaction
* Safer UX - users approve each step individually

The [preparePurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/blind-mint/preparepurchase) function returns [TransactionStep](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps) objects. Call [execute](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute) on each step to submit the transaction. The SDK automatically skips unnecessary steps (e.g., if approval is already granted).

For **server-side apps** or **agentic flows**, call [purchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/purchase) directly and the SDK handles all transactions sequentially without explicit step management.

### 2. Custom Execution with transactionData

Each `TransactionStep` includes a [transactionData](https://github.com/manifoldxyz/client-sdk/blob/main/packages/client-sdk/docs/sdk/transaction-steps/transactionData.md) field containing raw blockchain transaction data. This enables custom transaction execution for advanced use cases:

* **EIP-7702 Support**: Account abstraction and smart contract wallets
* **Custom Gas Management**: Implement your own gas strategies
* **Transaction Infrastructure**: Integrate with existing systems
* **Batch Transactions**: Combine multiple transactions
* **Custom Error Handling**: Implement retry logic

Example:

```typescript
// Prepare purchase normally
const preparedPurchase = await product.preparePurchase({
  userAddress: walletAddress,
  payload: { quantity: 1 }
});

// Execute with custom logic
for (const step of preparedPurchase.steps) {
  const { contractAddress, transactionData, value, gasEstimate } = step.transactionData;
  
  // Use your preferred Web3 library
  const txHash = await walletClient.sendTransaction({
    to: contractAddress,
    data: transactionData,
    value: value,
    gas: gasEstimate * 110n / 100n, // Custom gas buffer
  });
}
```

## Available Documentation

* [execute](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute) - Built-in execution method for transaction steps
* [TransactionData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactiondata) - Raw transaction data for custom execution


# execute

Handles the execution of a specific step.

**execute(params)** → [Receipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/receipt)

| Parameter     | Type                                                                            | Required | Description                               |
| ------------- | ------------------------------------------------------------------------------- | -------- | ----------------------------------------- |
| account       | [Account](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/account) | ✅        | Buyer’s account                           |
| confirmations | number                                                                          | ❌        | Number of confirmation blocks (Default 1) |

#### Returns: [Receipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/receipt)

#### Example:

```tsx
// account is created via createAccountEthers5 / createAccountViem
const receipts = []

for (const step of preparedPurchase.steps) {
  const receipt = await step.execute(account)
  receipts.push(receipt)

  console.log('Confirmed tx:', receipt.transactionReceipt.txHash)

  if (receipt.order) {
    receipt.order.items.forEach((item) => {
      console.log(`Minted token ${item.token.tokenId}`)
    })
  }
}
```


# InstanceData

| Field          | Type                                                                                                                                                                                                           | Required | Description                     |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------- |
| **id**         | string                                                                                                                                                                                                         | ✅        | Unique identifier (instance ID) |
| **creator**    | [Creator](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/creator)                                                                                                                                | ✅        | The creator of the product      |
| **publicData** | [EditionPublicData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editionpublicdata) \| [BlindMintPublicData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/blindmintpublicdata) | ✅        | The product data                |
| appId          | number                                                                                                                                                                                                         | ✅        | The appId of the product        |
| appName        | string                                                                                                                                                                                                         | ✅        | The app name of the product     |


# EditionProduct

| Field                                                                                                                 | Type                                                                                                  | Required | Description                                                 |
| --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------------- |
| onchainData                                                                                                           | [EditionOnchainData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/editiononchaindata) | ❌        | Product onchain data                                        |
| [**preparePurchase**](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/preparepurchase) | function                                                                                              | ✅        | Simulates purchase to check eligibility and get total cost. |
| [**purchase**](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/purchase)                        | function                                                                                              | ✅        | Make a purchase on the product                              |
| [fetchOnchainData](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/edition-product/fetchonchaindata)   | function                                                                                              | ✅        | Fetch on-chain data for this product                        |
| [getAllocations](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getallocations)                | function                                                                                              | ✅        | Check product eligibility quantity for a wallet address     |
| [getInventory](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getinventory)                    | [ProductInventory](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productinventory)     | ✅        | Get inventory of the product                                |
| [getRules](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getrules)                            | [ProductRule](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productrule)               | ✅        | Product specific rules                                      |
| [getProvenance](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/product/common/getprovenance)                  | [ProductProvenance](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/productprovenance)   | ✅        | Product provenance info                                     |


# EditionPublicData

| Field            | Type                                                                              | Required | Description                      |
| ---------------- | --------------------------------------------------------------------------------- | -------- | -------------------------------- |
| **title**        | string                                                                            | ✅        | Title of the product             |
| description      | string                                                                            | ❌        | Description of the product       |
| **asset**        | [Asset](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/asset)       | ✅        | Primary media of the product     |
| network          | number                                                                            | ✅        | The network the product is on    |
| contract         | [Contract](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/contract) | ✅        | The contract of the token        |
| extensionAddress | string                                                                            | ✅        | Extension address of the product |


# EditionOnchainData

| Field           | Type                                                                        | Required | Description                 |
| --------------- | --------------------------------------------------------------------------- | -------- | --------------------------- |
| totalSupply     | number                                                                      | ✅        | Total supply of the product |
| totalMinted     | number                                                                      | ✅        | Total token minted          |
| walletMax       | number                                                                      | ✅        | Max tokens per wallet       |
| startDate       | Date                                                                        | ✅        | Start drop date             |
| endDate         | Date                                                                        | ✅        | End drop date               |
| audienceType    | enum                                                                        | ✅        | `None`                      |
| cost            | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money) | ✅        | Cost of the product         |
| paymentReceiver | string                                                                      | ✅        | Receiver of mint payment    |


# BlindMintProduct

<table><thead><tr><th>Field</th><th width="195.76953125">Type</th><th>Required</th><th>Description</th></tr></thead><tbody><tr><td>onchainData</td><td><a href="blindmintonchaindata">BlindMintOnchainData</a></td><td>❌</td><td>Product onchain data</td></tr><tr><td><a href="../sdk/product/blind-mint/preparepurchase"><strong>preparePurchase</strong></a></td><td>function</td><td>✅</td><td>Simulates purchase to check eligibility and get total cost.</td></tr><tr><td><a href="../sdk/product/common/purchase"><strong>purchase</strong></a></td><td>function</td><td>✅</td><td>Make a purchase on the product</td></tr><tr><td><a href="../sdk/product/blind-mint/fetchonchaindata">fetchOnchainData</a></td><td>function</td><td>✅</td><td>Fetch on-chain data for this product</td></tr><tr><td><a href="../sdk/product/common/getallocations">getAllocations</a></td><td>function</td><td>✅</td><td>Check product eligibility quantity for a wallet address</td></tr><tr><td><a href="../sdk/product/common/getinventory">getInventory</a></td><td><a href="productinventory">ProductInventory</a></td><td>✅</td><td>Get inventory of the product</td></tr><tr><td><a href="../sdk/product/common/getrules">getRules</a></td><td><a href="productrule">ProductRule</a></td><td>✅</td><td>Product specific rules</td></tr><tr><td><a href="../sdk/product/common/getprovenance">getProvenance</a></td><td><a href="productprovenance">ProductProvenance</a></td><td>✅</td><td>Product provenance info</td></tr></tbody></table>


# BlindMintPublicData

<table><thead><tr><th>Field</th><th width="212.953125">Type</th><th>Required</th><th>Description</th></tr></thead><tbody><tr><td><strong>title</strong></td><td>string</td><td>✅</td><td>Title of the product</td></tr><tr><td>description</td><td>string</td><td>❌</td><td>Description of the product</td></tr><tr><td>network</td><td>number</td><td>✅</td><td>The network the product is on</td></tr><tr><td>contract</td><td><a href="contract">Contract</a></td><td>✅</td><td>The contract of the token</td></tr><tr><td>extensionAddress</td><td>string</td><td>✅</td><td>Extension address of the product</td></tr><tr><td>tierProbabilities</td><td><a href="blindminttierprobability">BlindMintTierProbability</a></td><td>✅</td><td>Tier and probability of each group</td></tr><tr><td>pool</td><td><a href="blindmintpool">BlindMintPool</a></td><td>✅</td><td>Pool of assets</td></tr></tbody></table>


# BlindMintOnchaindata

| Field           | Type                                                                        | Required | Description                            |
| --------------- | --------------------------------------------------------------------------- | -------- | -------------------------------------- |
| totalSupply     | number                                                                      | ✅        | Total supply of the product            |
| **totalMinted** | number                                                                      | ✅        | Total token minted                     |
| walletMax       | number                                                                      | ✅        | Max tokens per wallet                  |
| startDate       | Date                                                                        | ✅        | Start drop date                        |
| endDate         | Date                                                                        | ✅        | End drop date                          |
| audiencetype    | enum                                                                        | ✅        | `None`                                 |
| cost            | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money) | ✅        | Cost of the product                    |
| paymentReceiver | string                                                                      | ✅        | Receiver of mint payment               |
| tokenVariations | number                                                                      | ✅        | Number of asset variations             |
| startingTokenId | number                                                                      | ✅        | The starting tokenId of the asset pool |


# BlindMintTierProbability

| Field   | Type      | Required | Description                                       |
| ------- | --------- | -------- | ------------------------------------------------- |
| group   | string    | ✅        | Name of the tier group                            |
| indices | number\[] | ✅        | Asset indices belonging to the group              |
| rate    | number    |          | Probability chance in basis points (10000 = 100%) |


# BlindMintPool

| Field    | Type                                                                        | Required | Description                    |
| -------- | --------------------------------------------------------------------------- | -------- | ------------------------------ |
| index    | number                                                                      | ✅        | Index of the asset in the pool |
| metadata | [Asset](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/asset) | ✅        | Asset metadata                 |


# PreviewData

| Field         | Type                                                                              | Required | Description                          |
| ------------- | --------------------------------------------------------------------------------- | -------- | ------------------------------------ |
| title         | string                                                                            | ❌        | Product Title                        |
| description   | string                                                                            | ❌        | Product description                  |
| contract      | [Contract](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/contract) | ❌        | Contract associated with the product |
| thumbnail     | string                                                                            | ❌        | Thumbnail for the product            |
| payoutAddress | string                                                                            | ❌        | Payout wallet address of the product |
| network       | number                                                                            | ❌        | Network the product is on            |
| startDate     | Date                                                                              | ❌        | Start date for the product           |
| endDate       | Date                                                                              | ❌        | End date for the product             |
| price         | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money)       | ❌        | Price of the product                 |


# ProductMetadata

ProductMetadata

| Field       | Type   | Required | Description         |
| ----------- | ------ | -------- | ------------------- |
| name        | string | ✅        | Product name        |
| description | string | ❌        | Product description |


# ProductRule

| Field                   | Type   | Required | Description                                     |
| ----------------------- | ------ | -------- | ----------------------------------------------- |
| startDate               | Date   | ❌        | Start date (if not provided, start immediately) |
| endDate                 | Date   | ❌        | End date (not provided, never ends)             |
| **audienceRestriction** | enum   | ✅        | `allowlist`                                     |
| maxPerWallet            | number | ❌        | Number of allowed purchased per wallet          |


# ProductInventory

| Field          | Type   | Required | Description                                                       |
| -------------- | ------ | -------- | ----------------------------------------------------------------- |
| totalSupply    | number | ✅        | Total product supply (a value of `-1` indicates unlimited supply) |
| totalPurchased | number | ✅        | Total product purchased                                           |


# ProductProvenance

| Field     | Type                                                                              | Required | Description                                 |
| --------- | --------------------------------------------------------------------------------- | -------- | ------------------------------------------- |
| creator   | [Creator](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/creator)   | ✅        | Media url of the product                    |
| contract  | [Contract](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/contract) | ❌        | Link to the contract related to the product |
| token     | [Token](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/token)       | ❌        | Link to the token related to the product    |
| networkId | number                                                                            | ❌        | Network ID of the product (if applicable)   |


# TransactionStep

Individual transaction step within a purchase flow. Represents a single blockchain transaction that needs to be executed. Steps enable explicit user control over multi-transaction operations, following Web3 best practices for transparency and user consent.

## Fields

| Field                                                                                                                                    | Type                | Required | Description                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | -------- | ----------------------------------------------------------- |
| id                                                                                                                                       | string              | ✅        | Unique identifier for this step                             |
| name                                                                                                                                     | string              | ✅        | Human-readable step name (e.g., "Approve USDC", "Mint NFT") |
| type                                                                                                                                     | TransactionStepType | ✅        | Type of transaction: `'mint'` \| `'approve'`                |
| [transactionData](https://github.com/manifoldxyz/client-sdk/blob/main/packages/client-sdk/docs/sdk/transaction-steps/transactionData.md) | TransactionData     | ✅        | Raw transaction data for custom execution                   |
| [execute](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute)                                              | function            | ✅        | Executes this transaction step                              |
| description                                                                                                                              | string              | ❌        | Optional detailed description of what this step does        |
| cost                                                                                                                                     | object              | ❌        | Cost breakdown for this specific step                       |
| cost.native                                                                                                                              | Money               | ❌        | Native token cost (ETH, etc.)                               |
| cost.erc20s                                                                                                                              | Money\[]            | ❌        | ERC20 token costs                                           |

## Common Step Types

* **`approve`**: ERC20 token approval for payment
* **`mint`**: Actual NFT minting transaction

## Usage

Steps should be executed in order, as later steps may depend on earlier ones.

### Example: Iterating Through Steps

```typescript
// Execute steps manually for better UX
for (const step of preparedPurchase.steps) {
  console.log(`Executing: ${step.name}`);
  const receipt = await step.execute(adapter);
  console.log(`✓ ${step.name} complete: ${receipt.txHash}`);
}
```

## See Also

* [TransactionData](https://github.com/manifoldxyz/client-sdk/blob/main/packages/client-sdk/docs/sdk/transaction-steps/transactionData.md) - Raw transaction data for custom execution
* [execute](https://manifoldxyz.gitbook.io/manifold-client-sdk/sdk/transaction-steps/execute) - Built-in execution function
* [PreparedPurchase](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/preparedpurchase) - Parent structure containing steps


# PreparedPurchase

| Field               | Type                                                                                               | Required | Description                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------------------------------- |
| **cost**            | [Cost](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/cost)                          | ✅        | Purchase cost                                                          |
| **transactionData** | [TransactionData](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactiondata)    | ✅        | Transaction to execute                                                 |
| steps               | [TransactionStep](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactionstep)\[] | ✅        | Manual steps that need to be executed in order (For manual executions) |
| gasEstimate         | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money)                        | ✅        | Total gas estimated                                                    |


# TransactionCallbacks

| Field                                      | Type     | Required | Description                         |
| ------------------------------------------ | -------- | -------- | ----------------------------------- |
| onProgress (progress: TransactionProgress) | function | ❌        | Called when purchase status changes |


# TransactionProgress

| Field           | Type                                                                                               | Required | Description                                                                                                                                                                                              |
| --------------- | -------------------------------------------------------------------------------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| status          | enum                                                                                               | ✅        | `pending-approval`                                                                                                                                                                                       |
| steps           | [TransactionStep](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactionstep)\[] | ✅        | This is the full steps object as it’s returned by the api and enhanced with some additional properties on the client. Notably the step status has been updated as well as any errors have been attached. |
| **currentStep** | [TransactionStep](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactionstep)    | ✅        | We’ve conveniently pinpointed the current step that’s being processed and made it accessible in this callback.                                                                                           |
| receipts        | [Receipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/receipt)\[]                 | ❌        | A full list of all the transaction hashes that have been processed so far during execution.                                                                                                              |
| data            | object                                                                                             | ❌        | Additional context data                                                                                                                                                                                  |


# TransactionData

| Field           | Type   | Required | Description                            |
| --------------- | ------ | -------- | -------------------------------------- |
| contractAddress | string | ✅        | Target contract                        |
| transactionData | string | ✅        | Encoded calldata                       |
| gasEstimate     | BigInt | ✅        | Gas estimate of the transaction        |
| networkId       | number | ✅        | Network of transaction                 |
| value           | bigint | ✅        | The required value for the transaction |


# Order

Base order information shared across all purchase flows:

| Field            | Type                                                                      | Required | Description                                    |
| ---------------- | ------------------------------------------------------------------------- | -------- | ---------------------------------------------- |
| recipientAddress | string                                                                    | ✅        | Wallet address that receives the minted tokens |
| total            | [Cost](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/cost) | ✅        | Total cost for this purchase flow              |

`TokenOrder` extends `Order` and is returned when mint results are available:

| Field | Type                                                                                   | Required | Description                                                                |
| ----- | -------------------------------------------------------------------------------------- | -------- | -------------------------------------------------------------------------- |
| items | [OrderItem](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/orderitem)\[] | ✅        | Detailed list of minted tokens with quantities and per-item cost breakdown |


# OrderItem

Represents a minted token (or group of tokens) within a purchase order.

| Field    | Type                                                                        | Required | Description                               |
| -------- | --------------------------------------------------------------------------- | -------- | ----------------------------------------- |
| total    | [Cost](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/cost)   | ✅        | Cost allocation for this specific item    |
| token    | [Token](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/token) | ✅        | Token metadata (contract, media, tokenId) |
| quantity | number                                                                      | ✅        | Number of tokens represented by this item |


# Receipt

| Field              | Type                                                                                                  | Required | Description                                                            |
| ------------------ | ----------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------------------------------- |
| transactionReceipt | [TransactionReceipt](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/transactionreceipt) | ✅        | Normalized transaction metadata (hash, block number, gas usage)        |
| order              | [TokenOrder](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/order)                      | ❌        | Parsed mint results including tokens, quantities, and cost allocations |


# TransactionReceipt

| Field       | Type   | Required | Description                                 |
| ----------- | ------ | -------- | ------------------------------------------- |
| networkId   | number | ✅        | The network ID                              |
| txHash      | string | ✅        | The transaction hash                        |
| blockNumber | number | ✅        | The block number of the transaction         |
| gasUsed     | number | ✅        | The amount of gas used from the transaction |


# Account

| Field             | Type     | Required | Description                                        |
| ----------------- | -------- | -------- | -------------------------------------------------- |
| address           | string   | ✅        | The Ethereum Address                               |
| sendTransaction() | function | ✅        | Send the given transaction to the blockchain       |
| signMessage()     | function | ✅        | Sign the given message and return the signature    |
| signTypedData()   | function | ✅        | Sign the given typed data and return the signature |
| getNetworkId      | number   | ✅        | Currently connected networkId                      |
| switchNetwork     | function | ✅        | Switch to target network                           |


# Money

| Field        | Type   | Required | Description                                                                  |
| ------------ | ------ | -------- | ---------------------------------------------------------------------------- |
| value        | BigInt | ✅        | The raw amount                                                               |
| decimals     | number | ✅        | Number of decimal places for the currency                                    |
| currency     | string | ✅        | `ETH`                                                                        |
| erc20        | string | ✅        | “0x0000000000000000000000000000000000000000” for native ETH or ERC20 address |
| symbol       | string | ✅        | `ETH`                                                                        |
| name         | string | ✅        | Name of the currency                                                         |
| formatted    | string | ✅        | The formatted amount (Ex: “0.1 ETH” )                                        |
| formattedUSD | string | ✅        | The formatted amount in USD                                                  |


# Cost

| Field    | Type                                                                        | Required | Description                   |
| -------- | --------------------------------------------------------------------------- | -------- | ----------------------------- |
| total    | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money) | ✅        | Total cost including all fees |
| subtotal | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money) | ✅        | Cost excluding fees           |
| fees     | [Money](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/money) | ✅        | Platform fees                 |


# AppType

| Field       | Type   | Required | Description          |
| ----------- | ------ | -------- | -------------------- |
| Edition     | string | ✅        | Edition app type     |
| Burn Redeem | string | ✅        | Burn Redeem app type |
| Blind Mint  | string | ✅        | Blind Mint app type  |


# TokenRequirement

| Field         | Type                                                                                                         | Required | Description                 |
| ------------- | ------------------------------------------------------------------------------------------------------------ | -------- | --------------------------- |
| items         | [TokenItemRequirement](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/tokenitemrequirement)\[] | ✅        | List of eligible token sets |
| requiredCount | number                                                                                                       | ✅        | Number of tokens required   |


# TokenItemRequirement

| Field           | Type      | Required | Description                     |
| --------------- | --------- | -------- | ------------------------------- |
| quantity        | number    | ✅        | Required quantity of tokens     |
| burnSpec        | enum      | ✅        | `manifold`                      |
| tokenSpec       | enum      | ✅        | `erc721`                        |
| tokenIds        | string\[] | ❌        | list of required tokenIds       |
| maxTokenId      | string    | ❌        | Max tokenId range               |
| minTokenId      | string    | ❌        | Min tokenId range               |
| contractAddress | string    | ✅        | The required contract address   |
| merkleRoot      | string    | ❌        | For allowlist token requirement |
| validationType  | enum      | ✅        | `contract`                      |


# Creator

| Field   | Type   | Required | Description                                  |
| ------- | ------ | -------- | -------------------------------------------- |
| id      | string | ✅        | identifier of the workspace                  |
| slug    | string | ✅        | Slug of workspace                            |
| address | string | ✅        | The Ethereum wallet address of the workspace |
| name    | string | ❌        | Name of workspace                            |


# Identity

| Field             | Type   | Required | Description                 |
| ----------------- | ------ | -------- | --------------------------- |
| walletAddress     | string | ✅        | The Ethereum wallet address |
| twitterUsername   | string | ❌        | The user X username         |
| instagramUsername | string | ❌        | The user instagram username |


# Contract

| Field     | Type                                                                               |   | Description                       |
| --------- | ---------------------------------------------------------------------------------- | - | --------------------------------- |
| networkId | number                                                                             | ✅ | The Ethereum Network              |
| address   | string                                                                             | ✅ | The contract address              |
| explorer  | [Explorer](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/explorers) | ✅ | The explorer urls of the contract |
| name      | string                                                                             |   |                                   |
| symbol    | string                                                                             |   |                                   |
| spec      | enum                                                                               | ✅ | `erc1155`                         |


# Token

| Field    | Type                                                                               |   | Description                    |
| -------- | ---------------------------------------------------------------------------------- | - | ------------------------------ |
| contract | [Contract](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/contract)  | ✅ |                                |
| tokenId  | string                                                                             | ✅ | The token ID                   |
| explorer | [Explorer](https://manifoldxyz.gitbook.io/manifold-client-sdk/reference/explorers) | ✅ | The explorer urls of the token |


# Explorers

| Field        | Type   | Required | Description                            |
| ------------ | ------ | -------- | -------------------------------------- |
| etherscanUrl | string | ✅        | Link to Etherscan Explorer hosted site |
| manifoldUrl  | string | ❌        | Link to Manifold hosted site           |
| openseaUrl   | string | ❌        | Link to Opensea hosted site            |


# Asset

| Field            | Type   | Required | Description                          |
| ---------------- | ------ | -------- | ------------------------------------ |
| name             | string | ✅        |                                      |
| description      | string | ❌        |                                      |
| attributes       | object | ❌        | Extra attributes (key/value)         |
| image            | string | ✅        | Image url of the product             |
| imagePreview     | string | ❌        | Preview image url of the product     |
| animation        | string | ❌        | Animation url of the product         |
| animationPreview | string | ❌        | Animation preview url of the product |


# ClientSDKError

Base error class with typed error codes.

| Field   | Type   | Required |
| ------- | ------ | -------- |
| code    | string | ✅        |
| message | string | ✅        |
| details | object | ❌        |

**Error Codes**

| Code                  | Description                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Network Errors**    |                                                                                                                     |
| UNSUPPORTED\_NETWORK  | Unsupported network                                                                                                 |
| **Data Errors**       |                                                                                                                     |
| NOT\_FOUND            | Resource not found                                                                                                  |
| INVALID\_INPUT        | Invalid parameters                                                                                                  |
| MISSING\_TOKENS       | Missing required tokens to purchase                                                                                 |
| UNSUPPORTED\_TYPE     | Unsupported product type (Only support the following types: AppType.Edition, AppType.BlindMint) |
| **Blockchain Errors** |                                                                                                                     |
| ESTIMATION\_FAILED    | Can’t estimate gas                                                                                                  |
| TRANSACTION\_FAILED   | Transaction reverted                                                                                                |
| LEDGER\_ERROR         | Ledger wallet error                                                                                                 |
| TRANSACTION\_REVERTED | Transaction revert                                                                                                  |
| TRANSACTION\_REJECTED | User rejected                                                                                                       |
| INSUFFICIENT\_FUNDS   | Wallet does not have the required funds                                                                             |
| **Permission Errors** |                                                                                                                     |
| NOT\_ELIGIBLE         | Not eligible to purchase                                                                                            |
| SOLD\_OUT             | Product sold out                                                                                                    |
| LIMIT\_REACHED        | Limit reach for wallet                                                                                              |
| ENDED                 | Product not available anymore                                                                                       |
| NOT\_STARTED          | Not started, come back late                                                                                         |


