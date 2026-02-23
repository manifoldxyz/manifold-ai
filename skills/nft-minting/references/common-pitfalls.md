# Common Pitfalls

Known mistakes that AI agents and developers frequently make with the Manifold SDK.

## LLM-Specific Traps

### ❌ Fabricating method signatures

**Wrong:** Inventing parameters that don't exist.

```typescript
// ❌ WRONG — 'address' is not a valid parameter
await product.preparePurchase({
  address: '0x...',
  payload: { quantity: 1 },
});

// ✅ CORRECT — use 'userAddress'
await product.preparePurchase({
  userAddress: '0x...',
  payload: { quantity: 1 },
});
```

### ❌ Using `recipientAddress` instead of `userAddress`

`userAddress` is the wallet paying for the transaction. `recipientAddress` is an optional different recipient.

```typescript
// ❌ WRONG
await product.preparePurchase({
  recipientAddress: '0x...', // This is optional, not the primary param
  payload: { quantity: 1 },
});

// ✅ CORRECT
await product.preparePurchase({
  userAddress: '0x...',         // Who pays
  recipientAddress: '0x...',    // Optional: who receives (defaults to userAddress)
  payload: { quantity: 1 },
});
```

### ❌ Accessing order.receipts instead of transactionReceipt

The `purchase()` return shape is:

```typescript
// ❌ WRONG
const txHash = order.receipts[0]?.txHash;

// ✅ CORRECT
const txHash = result.transactionReceipt.txHash;
const order = result.order; // Contains minted token details
```

### ❌ Not using type guards

```typescript
// ❌ WRONG — assumes product type
const prepared = await (product as EditionProduct).preparePurchase({ ... });

// ✅ CORRECT — verify first
if (isEditionProduct(product)) {
  const prepared = await product.preparePurchase({ ... });
}
```

### ❌ Skipping status checks

```typescript
// ❌ WRONG — goes straight to prepare, wastes RPC calls on inactive products
const prepared = await product.preparePurchase({ ... });

// ✅ CORRECT — check first
const status = await product.getStatus();
if (status !== 'active') {
  throw new Error(`Product is ${status}`);
}
const prepared = await product.preparePurchase({ ... });
```

## SDK-Specific Gotchas

### createPublicProviderViem takes a Record, not a single client

```typescript
// ❌ WRONG
const publicProvider = createPublicProviderViem(publicClient);

// ✅ CORRECT — pass a Record<chainId, PublicClient>
const publicProvider = createPublicProviderViem({ [chainId]: publicClient });
```

### Account is separate from PublicProvider

The SDK separates read operations (PublicProvider) from write operations (Account).

```typescript
// PublicProvider → for reading chain data (required for createClient)
const publicProvider = createPublicProviderViem({ ... });
const client = createClient({ publicProvider });

// Account → for signing transactions (required for purchase)
const account = createAccountViem({ walletClient });
```

### Gas buffer uses multiplier, not percentage

```typescript
// ❌ WRONG — 25 means 2500% gas buffer
gasBuffer: { multiplier: 25 }

// ✅ CORRECT — 0.25 means 25% additional gas
gasBuffer: { multiplier: 0.25 }
```

### Money.value is bigint, Money.raw is BN

Use `.formatted` for display, `.value` for calculations:

```typescript
// ❌ WRONG — mixing types
const total = cost.total.native.raw + someOtherBigint;

// ✅ CORRECT
console.log(cost.total.native.formatted);     // "0.05 ETH"
console.log(cost.total.native.formattedUSD);  // "150.00"
```

### Don't pass account to createClient

```typescript
// ❌ WRONG — createClient doesn't take account
const client = createClient({ publicProvider, account });

// ✅ CORRECT — account is used in preparePurchase and purchase
const client = createClient({ publicProvider });
const prepared = await product.preparePurchase({ ..., account });
const result = await product.purchase({ account, preparedPurchase: prepared });
```

## Framework Gotchas

### Next.js: SDK is client-side only

The SDK requires browser APIs (or a wallet) for transactions. Mark components with `'use client'`:

```tsx
'use client'; // Required!

import { createClient } from '@manifoldxyz/client-sdk';
```

### Private keys in server-side code only

For minting bots, keep private keys server-side:

```typescript
// ❌ WRONG — exposed to browser
const key = process.env.NEXT_PUBLIC_PRIVATE_KEY;

// ✅ CORRECT — server-only env var
const key = process.env.WALLET_PRIVATE_KEY; // No NEXT_PUBLIC_ prefix
```

## Reference Sources

When in doubt, always verify against:

1. **Official SDK docs:** https://docs.manifold.xyz/client-sdk/
2. **Full LLM docs:** https://manifold-1.gitbook.io/manifold-client-sdk/llms-full.txt
3. **SDK source code:** https://github.com/manifoldxyz/client-sdk
4. **SDK npm package:** `@manifoldxyz/client-sdk` type definitions
