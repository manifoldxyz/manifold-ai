# Purchase Flow

The SDK uses a **two-step purchase pattern**: prepare, then execute.

## Overview

```
getProduct() → preparePurchase() → purchase()
```

1. **Prepare** — validates eligibility, calculates costs, generates transaction data
2. **Execute** — sends transaction(s) to the blockchain

## Step 1: Prepare Purchase

`preparePurchase` does all validation and returns a `PreparedPurchase` with cost breakdown and transaction steps.

### Edition Product

```typescript
const prepared = await product.preparePurchase({
  userAddress: '0x...',          // Wallet funding the transaction
  recipientAddress: '0x...',     // Optional: different recipient (defaults to userAddress)
  payload: {
    quantity: 2,                 // Number to mint
  },
  account,                       // Optional: pass account to check balances
  gasBuffer: {                   // Optional: gas buffer
    multiplier: 0.25,            // 25% buffer above estimate
  },
});
```

### Blind Mint Product

```typescript
const prepared = await product.preparePurchase({
  userAddress: '0x...',
  payload: {
    quantity: 1,
  },
  account,
  gasBuffer: { multiplier: 0.25 },
});
```

### PreparedPurchase Object

```typescript
interface PreparedPurchase {
  cost: Cost;                    // Full cost breakdown
  steps: TransactionStep[];      // Transaction steps to execute
  isEligible: boolean;           // Whether the wallet can purchase
}
```

### Cost Breakdown

```typescript
interface Cost {
  totalUSD: string;              // Total cost in USD
  total: {
    native: Money;               // Native token cost (ETH)
    erc20s: Money[];             // ERC-20 token costs
  };
  breakdown: {
    product: Money;              // Creator's price
    platformFee: Money;          // Manifold platform fee
  };
}
```

### Money Object

The `Money` class provides formatted values:

```typescript
cost.total.native.formatted    // "0.05 ETH"
cost.total.native.formattedUSD // "150.00"
cost.total.native.value        // bigint (raw wei)
cost.total.native.symbol       // "ETH"
cost.total.native.decimals     // 18
cost.total.native.isPositive() // true
cost.total.native.isERC20()    // false
```

## Confirmation Before Execution

**Always display cost and get user confirmation before executing a purchase.** After `preparePurchase` returns, show the cost details and wait for explicit approval:

```typescript
// Display cost details to the user
console.log(`Cost: ${prepared.cost.total.native.formatted}`);
console.log(`Network: ${networkId} | Quantity: ${quantity}`);
// Agent must confirm with user before proceeding
```

Do not call `purchase()` or `step.execute()` until the user has confirmed.

## Step 2: Execute Purchase

### Simple: Use `product.purchase()`

For most cases, let the SDK handle all steps:

```typescript
const result = await product.purchase({
  account,                       // IAccount adapter
  preparedPurchase: prepared,    // From preparePurchase
  confirmations: 1,              // Optional: blocks to wait (default 1)
});

console.log(`TX: ${result.transactionReceipt.txHash}`);

// Edition products return order details with minted tokens
if (result.order) {
  result.order.items.forEach((item) => {
    console.log(`Token ${item.token.tokenId} x${item.quantity}`);
    console.log(`Image: ${item.token.media.image}`);
  });
}
```

### Advanced: Execute Steps Manually

For UI that shows each step (e.g., "Approve USDC" → "Mint"):

```typescript
for (const step of prepared.steps) {
  console.log(`Executing: ${step.name} — ${step.description}`);
  const receipt = await step.execute(account);
  console.log(`Done: ${receipt.transactionReceipt.txHash}`);
}
```

See `references/transaction-steps.md` for details.

## Complete Example

```typescript
import 'dotenv/config';
import {
  createClient,
  createAccountViem,
  createPublicProviderViem,
  isEditionProduct,
} from '@manifoldxyz/client-sdk';
import { createPublicClient, createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

// Setup
const publicClient = createPublicClient({ chain: base, transport: http(process.env.RPC_URL!) });
const publicProvider = createPublicProviderViem({ [base.id]: publicClient });
const client = createClient({ publicProvider });

// Fetch & validate
const product = await client.getProduct(process.env.INSTANCE_ID!);
if (!isEditionProduct(product)) throw new Error('Not an edition');

const status = await product.getStatus();
if (status !== 'active') throw new Error(`Product is ${status}`);

// Create account
const wallet = privateKeyToAccount(process.env.WALLET_PRIVATE_KEY! as `0x${string}`);
const walletClient = createWalletClient({
  account: wallet,
  chain: base,
  transport: http(process.env.RPC_URL!),
});
const account = createAccountViem({ walletClient });
const address = await account.getAddress();

// Check eligibility
const allocations = await product.getAllocations({ recipientAddress: address });
if (!allocations.isEligible) throw new Error(allocations.reason || 'Not eligible');

// Prepare
const prepared = await product.preparePurchase({
  userAddress: address,
  payload: { quantity: 1 },
  account,
});

console.log(`Cost: ${prepared.cost.total.native.formatted}`);
console.log(`Steps: ${prepared.steps.length}`);

// Execute
const result = await product.purchase({ account, preparedPurchase: prepared });
console.log(`Minted! TX: ${result.transactionReceipt.txHash}`);
```

## Validation Errors from preparePurchase

`preparePurchase` throws `ClientSDKError` for:

| Error Code | Meaning |
|-----------|---------|
| `NOT_STARTED` | Sale hasn't started yet |
| `ENDED` | Sale has ended |
| `SOLD_OUT` | No supply remaining |
| `NOT_ELIGIBLE` | Wallet not on allowlist or doesn't meet requirements |
| `INVALID_INPUT` | Bad address, quantity exceeds allocation, etc. |
| `INSUFFICIENT_FUNDS` | Wallet doesn't have enough tokens |
