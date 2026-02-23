# Building a Minting Bot

Headless server-side minting using a private key. No browser required.

**Official examples:**
- [Edition bot](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/minting-bot)
- [Blind Mint bot](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/blindmint/minting-bot)

## Project Setup

```bash
mkdir my-mint-bot && cd my-mint-bot
npm init -y
npm install @manifoldxyz/client-sdk dotenv
npm install -D typescript tsx @types/node
npx tsc --init
```

Create `.env`:

```env
INSTANCE_ID=your_instance_id
NETWORK_ID=11155111
RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
WALLET_PRIVATE_KEY=your_private_key_here
MINT_QUANTITY=1
```

## Using Viem (Recommended)

Create `src/index.ts`:

```typescript
import 'dotenv/config';
import {
  createClient,
  createAccountViem,
  createPublicProviderViem,
  isEditionProduct,
  isBlindMintProduct,
} from '@manifoldxyz/client-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { sepolia } from 'viem/chains';

function getEnv(name: string): string {
  const value = process.env[name];
  if (!value) throw new Error(`Missing env: ${name}`);
  return value;
}

async function main() {
  const instanceId = getEnv('INSTANCE_ID');
  const rpcUrl = getEnv('RPC_URL');
  const networkId = Number(getEnv('NETWORK_ID'));
  const privateKey = getEnv('WALLET_PRIVATE_KEY');
  const quantity = Number(process.env.MINT_QUANTITY ?? '1');

  // Setup provider
  const publicClient = createPublicClient({
    chain: sepolia,
    transport: http(rpcUrl),
  });
  const publicProvider = createPublicProviderViem({ [networkId]: publicClient });
  const client = createClient({ publicProvider });

  // Fetch product
  console.log(`📦 Fetching product ${instanceId}...`);
  const product = await client.getProduct(instanceId);

  if (!isEditionProduct(product) && !isBlindMintProduct(product)) {
    throw new Error('Unsupported product type');
  }

  // Check status
  const status = await product.getStatus();
  if (status !== 'active') {
    throw new Error(`Product is ${status}`);
  }

  // Setup wallet
  const key = privateKey.startsWith('0x') ? privateKey : `0x${privateKey}`;
  const wallet = privateKeyToAccount(key as `0x${string}`);
  const walletClient = createWalletClient({
    account: wallet,
    chain: sepolia,
    transport: http(rpcUrl),
  });
  const account = createAccountViem({ walletClient });
  const address = await account.getAddress();
  console.log(`👤 Wallet: ${address}`);

  // Check eligibility
  const allocations = await product.getAllocations({ recipientAddress: address });
  if (!allocations.isEligible) {
    throw new Error(allocations.reason || 'Not eligible');
  }

  // Prepare
  console.log('🧪 Preparing purchase...');
  const prepared = await product.preparePurchase({
    userAddress: address,
    payload: { quantity },
    account,
  });

  const cost = prepared.cost.total.native;
  console.log(`💰 Cost: ${cost.formatted} (~$${prepared.cost.totalUSD})`);
  console.log(`📋 Steps: ${prepared.steps.length}`);

  // Execute
  console.log('🚀 Executing purchase...');
  const result = await product.purchase({
    account,
    preparedPurchase: prepared,
  });

  console.log(`🎉 TX: ${result.transactionReceipt.txHash}`);

  if (result.order) {
    result.order.items.forEach((item, i) => {
      console.log(`   Token ${i + 1}: #${item.token.tokenId} x${item.quantity}`);
    });
  }
}

main().catch((error) => {
  console.error('❌ Failed:', error.message);
  process.exit(1);
});
```

Run: `npx tsx src/index.ts`

## Using Ethers v5

```typescript
import 'dotenv/config';
import {
  createClient,
  createAccountEthers5,
  createPublicProviderEthers5,
  isEditionProduct,
} from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

async function main() {
  const rpcUrl = process.env.RPC_URL!;
  const networkId = Number(process.env.NETWORK_ID!);

  const provider = new ethers.providers.JsonRpcProvider(rpcUrl);
  const publicProvider = createPublicProviderEthers5({ [networkId]: provider });
  const client = createClient({ publicProvider });

  const product = await client.getProduct(process.env.INSTANCE_ID!);
  if (!isEditionProduct(product)) throw new Error('Not an edition');

  const wallet = new ethers.Wallet(process.env.WALLET_PRIVATE_KEY!, provider);
  const account = createAccountEthers5({ wallet });
  const address = await wallet.getAddress();

  const prepared = await product.preparePurchase({
    userAddress: address,
    payload: { quantity: 1 },
    account,
  });

  const result = await product.purchase({
    account,
    preparedPurchase: prepared,
  });

  console.log(`TX: ${result.transactionReceipt.txHash}`);
}

main().catch(console.error);
```

## Package.json Scripts

```json
{
  "scripts": {
    "start": "tsx src/index.ts",
    "dev": "tsx watch src/index.ts"
  }
}
```

## Best Practices

- **Never commit private keys** — use `.env` files and add `.env` to `.gitignore`
- **Test on Sepolia first** — use testnet before mainnet
- **Implement retry logic** — network errors can be transient
- **Monitor gas prices** — for mainnet, check gas before sending
- **Log everything** — transaction hashes, errors, timestamps
- **Handle graceful shutdown** — don't leave pending transactions

## Retry Pattern

```typescript
async function mintWithRetry(maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await product.purchase({ account, preparedPurchase: prepared });
      return result;
    } catch (error) {
      if (error instanceof ClientSDKError) {
        // Don't retry validation errors
        if ([ErrorCode.SOLD_OUT, ErrorCode.ENDED, ErrorCode.NOT_ELIGIBLE].includes(error.code)) {
          throw error;
        }
      }
      if (attempt === maxRetries) throw error;
      console.log(`Attempt ${attempt} failed, retrying in ${attempt * 2}s...`);
      await new Promise((r) => setTimeout(r, attempt * 2000));
    }
  }
}
```
