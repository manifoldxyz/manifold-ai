# Error Handling & Common Pitfalls

## ClientSDKError

All SDK errors use typed error codes:

```typescript
import { ClientSDKError, ErrorCode } from '@manifoldxyz/client-sdk';

try {
  const prepared = await product.preparePurchase({ ... });
  const result = await product.purchase({ account, preparedPurchase: prepared });
} catch (error) {
  if (error instanceof ClientSDKError) {
    console.log(`[${error.code}] ${error.message}`);
    console.log(error.details); // Additional context (varies by error)
  }
}
```

## Error Codes

### Validation (from preparePurchase)

| Code | Meaning | Recovery |
|------|---------|----------|
| `INVALID_INPUT` | Bad address, quantity, or instance ID | Fix input |
| `NOT_FOUND` | Instance ID doesn't exist | Verify ID |
| `UNSUPPORTED_PRODUCT_TYPE` | Not Edition or BlindMint | SDK only supports these two |
| `NOT_STARTED` | Sale start date not reached | Show countdown |
| `ENDED` | Sale end date passed | Inform user |
| `SOLD_OUT` | No supply remaining | Inform user |
| `NOT_ELIGIBLE` | Not on allowlist / doesn't meet criteria | Try different wallet |
| `INSUFFICIENT_FUNDS` | Balance too low | Add funds |

### Transaction (from purchase / step.execute)

| Code | Meaning | Recovery |
|------|---------|----------|
| `TRANSACTION_FAILED` | On-chain revert | Check details, retry |
| `TRANSACTION_REVERTED` | EVM revert | Check contract state |
| `TRANSACTION_REJECTED` | User rejected in wallet | Prompt retry |
| `LEDGER_ERROR` | Ledger wallet issue | Enable blind signing |
| `API_ERROR` | Backend API failed | Retry with backoff |

For `TRANSACTION_FAILED`, `error.details` may include `receipts` from completed steps.

## React Error Pattern

```tsx
const [error, setError] = useState<string | null>(null);

const handleMint = async () => {
  setError(null);
  try {
    const status = await product.getStatus();
    if (status !== 'active') { setError(`Drop is ${status}`); return; }

    const alloc = await product.getAllocations({ recipientAddress: address });
    if (!alloc.isEligible) { setError(alloc.reason || 'Not eligible'); return; }

    const prepared = await product.preparePurchase({
      userAddress: address, payload: { quantity: 1 }, account,
    });
    await product.purchase({ account, preparedPurchase: prepared });
  } catch (err) {
    setError(err instanceof ClientSDKError ? err.message : 'Unexpected error');
  }
};
```

## Common Pitfalls

### Wrong parameter names

```typescript
// ❌ 'address' doesn't exist
await product.preparePurchase({ address: '0x...', payload: { quantity: 1 } });

// ✅ Use 'userAddress' (payer) + optional 'recipientAddress' (receiver)
await product.preparePurchase({ userAddress: '0x...', payload: { quantity: 1 } });
```

### Wrong return shape from purchase()

```typescript
// ❌ No .receipts array
const txHash = order.receipts[0]?.txHash;

// ✅ Direct properties
const txHash = result.transactionReceipt.txHash;
const order = result.order; // minted token details
```

### Missing type guards

```typescript
// ❌ Unsafe cast
const prepared = await (product as EditionProduct).preparePurchase({ ... });

// ✅ Guard first
if (isEditionProduct(product)) {
  const prepared = await product.preparePurchase({ ... });
}
```

### Provider factory takes a Record, not a single client

```typescript
// ❌ Wrong
const publicProvider = createPublicProviderViem(publicClient);

// ✅ Pass Record<chainId, PublicClient>
const publicProvider = createPublicProviderViem({ [chainId]: publicClient });
```

### Account is not passed to createClient

```typescript
// ❌ createClient doesn't take account
const client = createClient({ publicProvider, account });

// ✅ Account goes to preparePurchase and purchase
const client = createClient({ publicProvider });
```

### Gas buffer is a multiplier, not a percentage

```typescript
// ❌ 25 means 2500% buffer
gasBuffer: { multiplier: 25 }

// ✅ 0.25 means 25% above estimate
gasBuffer: { multiplier: 0.25 }
```

### Next.js: SDK requires 'use client'

```tsx
'use client'; // Required — SDK needs browser/wallet APIs
import { createClient } from '@manifoldxyz/client-sdk';
```

### Private keys: server-only

```typescript
// ❌ Exposed to browser
const key = process.env.NEXT_PUBLIC_PRIVATE_KEY;

// ✅ Server-only env var (no NEXT_PUBLIC_ prefix)
const key = process.env.WALLET_PRIVATE_KEY;
```

## Authoritative Sources

When in doubt, verify against:
- SDK docs: https://docs.manifold.xyz/client-sdk/
- LLM docs: https://manifold-1.gitbook.io/manifold-client-sdk/llms-full.txt
- SDK source: https://github.com/manifoldxyz/client-sdk
