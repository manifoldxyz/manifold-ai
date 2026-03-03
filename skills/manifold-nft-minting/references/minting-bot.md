# Minting Bot

Headless server-side minting with a private key. No browser required.

**Official examples:**
- [Edition bot](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/edition/minting-bot)
- [Blind Mint bot](https://github.com/manifoldxyz/client-sdk/tree/main/packages/examples/blindmint/minting-bot)

## Dependencies

```bash
npm install @manifoldxyz/client-sdk dotenv viem
# or for ethers v5: npm install @manifoldxyz/client-sdk dotenv ethers@5
```

## Environment Variables

```env
INSTANCE_ID=your_instance_id
NETWORK_ID=11155111
RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
WALLET_PRIVATE_KEY=your_private_key
MINT_QUANTITY=1
```

## Complete Bot (Viem)

```typescript
import 'dotenv/config';
import {
  createClient, createAccountViem, createPublicProviderViem,
  isEditionProduct, isBlindMintProduct,
} from '@manifoldxyz/client-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { sepolia } from 'viem/chains';

async function main() {
  const networkId = Number(process.env.NETWORK_ID!);
  const rpcUrl = process.env.RPC_URL!;
  const quantity = Number(process.env.MINT_QUANTITY ?? '1');

  // Setup
  const publicProvider = createPublicProviderViem({
    [networkId]: createPublicClient({ chain: sepolia, transport: http(rpcUrl) }),
  });
  const client = createClient({ publicProvider });

  // Fetch + validate
  const product = await client.getProduct(process.env.INSTANCE_ID!);
  if (!isEditionProduct(product) && !isBlindMintProduct(product)) {
    throw new Error('Unsupported product type');
  }

  const status = await product.getStatus();
  if (status !== 'active') throw new Error(`Product is ${status}`);

  // Wallet
  const key = process.env.WALLET_PRIVATE_KEY!;
  const wallet = privateKeyToAccount((key.startsWith('0x') ? key : `0x${key}`) as `0x${string}`);
  const walletClient = createWalletClient({
    account: wallet, chain: sepolia, transport: http(rpcUrl),
  });
  const account = createAccountViem({ walletClient });
  const address = await account.getAddress();

  // Check eligibility
  const alloc = await product.getAllocations({ recipientAddress: address });
  if (!alloc.isEligible) throw new Error(alloc.reason || 'Not eligible');

  // Prepare
  const prepared = await product.preparePurchase({
    userAddress: address, payload: { quantity }, account,
  });

  // Confirm before executing
  console.log(`\nReady to mint:`);
  console.log(`  Cost: ${prepared.cost.total.native.formatted}`);
  console.log(`  Network: ${networkId} | Quantity: ${quantity}`);
  const readline = await import('readline');
  const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
  const answer = await new Promise<string>((resolve) =>
    rl.question('Proceed with purchase? (yes/no): ', resolve)
  );
  rl.close();
  if (answer.toLowerCase() !== 'yes') {
    console.log('Purchase cancelled.');
    process.exit(0);
  }

  // Execute
  const result = await product.purchase({ account, preparedPurchase: prepared });
  console.log(`TX: ${result.transactionReceipt.txHash}`);

  result.order?.items.forEach((item, i) => {
    console.log(`Token ${i + 1}: #${item.token.tokenId} x${item.quantity}`);
  });
}

main().catch((e) => { console.error('Failed:', e.message); process.exit(1); });
```

## Ethers v5 Variant

Replace the setup and wallet sections:

```typescript
import { createPublicProviderEthers5, createAccountEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const provider = new ethers.providers.JsonRpcProvider(rpcUrl);
const publicProvider = createPublicProviderEthers5({ [networkId]: provider });
const wallet = new ethers.Wallet(process.env.WALLET_PRIVATE_KEY!, provider);
const account = createAccountEthers5({ wallet });
```

The rest of the flow (getProduct → getStatus → preparePurchase → purchase) is identical.

## Retry Pattern

```typescript
async function mintWithRetry(maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await product.purchase({ account, preparedPurchase: prepared });
    } catch (error) {
      // Don't retry terminal errors
      if (error instanceof ClientSDKError &&
          [ErrorCode.SOLD_OUT, ErrorCode.ENDED, ErrorCode.NOT_ELIGIBLE].includes(error.code)) {
        throw error;
      }
      if (attempt === maxRetries) throw error;
      await new Promise((r) => setTimeout(r, attempt * 2000));
    }
  }
}
```

## Best Practices

- **Never commit private keys** — `.env` + `.gitignore`
- **Test on Sepolia first** — switch to mainnet when ready
- **Implement retry logic** — network errors are transient
- **Log transaction hashes** — for debugging and auditing
