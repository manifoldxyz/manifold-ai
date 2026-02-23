# Error Handling

All SDK errors are thrown as `ClientSDKError` with typed error codes.

## ClientSDKError

```typescript
import { ClientSDKError, ErrorCode } from '@manifoldxyz/client-sdk';

try {
  const prepared = await product.preparePurchase({ ... });
  const result = await product.purchase({ account, preparedPurchase: prepared });
} catch (error) {
  if (error instanceof ClientSDKError) {
    switch (error.code) {
      case ErrorCode.NOT_STARTED:
        console.log('Sale hasn\'t started yet');
        break;
      case ErrorCode.ENDED:
        console.log('Sale has ended');
        break;
      case ErrorCode.SOLD_OUT:
        console.log('Sold out');
        break;
      case ErrorCode.NOT_ELIGIBLE:
        console.log('Not eligible:', error.message);
        break;
      case ErrorCode.INSUFFICIENT_FUNDS:
        console.log('Insufficient funds:', error.message);
        break;
      case ErrorCode.TRANSACTION_FAILED:
        console.log('Transaction failed:', error.message);
        // error.details may contain receipts from completed steps
        break;
      default:
        console.log(`Error [${error.code}]: ${error.message}`);
    }
  } else {
    console.error('Unexpected error:', error);
  }
}
```

## Error Codes

### Validation Errors (from preparePurchase)

| Code | When | Recovery |
|------|------|----------|
| `INVALID_INPUT` | Bad address, quantity out of range, invalid instance ID | Fix the input |
| `NOT_FOUND` | Instance ID doesn't exist | Verify the instance ID |
| `UNSUPPORTED_PRODUCT_TYPE` | Product type not supported by SDK | Only Edition and BlindMint are supported |
| `NOT_STARTED` | Sale start date hasn't been reached | Wait or show countdown |
| `ENDED` | Sale end date has passed | Inform user |
| `SOLD_OUT` | All supply minted | Inform user |
| `NOT_ELIGIBLE` | Wallet not on allowlist / doesn't meet criteria | Check with different wallet |
| `INSUFFICIENT_FUNDS` | Wallet balance too low | Add funds |

### Transaction Errors (from purchase / step.execute)

| Code | When | Recovery |
|------|------|----------|
| `TRANSACTION_FAILED` | On-chain transaction reverted | Check error details, retry |
| `TRANSACTION_REVERTED` | Transaction was reverted by the EVM | Check contract state |
| `TRANSACTION_REJECTED` | User rejected in wallet | Prompt user to try again |
| `LEDGER_ERROR` | Ledger wallet issue | Enable blind signing on Ledger |
| `API_ERROR` | Backend API call failed | Retry with backoff |

### Error Details

`ClientSDKError` may include a `details` object with additional context:

```typescript
catch (error) {
  if (error instanceof ClientSDKError) {
    console.log(error.code);      // ErrorCode enum value
    console.log(error.message);   // Human-readable message
    console.log(error.details);   // Additional context (varies by error)

    // For TRANSACTION_FAILED, details may include completed step receipts
    if (error.code === ErrorCode.TRANSACTION_FAILED && error.details?.receipts) {
      error.details.receipts.forEach((receipt) => {
        console.log(`Completed step TX: ${receipt.txHash}`);
      });
    }
  }
}
```

## Best Practices

1. **Always wrap purchase flows in try/catch** — both `preparePurchase` and `purchase` can throw
2. **Check `getStatus()` before preparing** — avoids unnecessary RPC calls
3. **Check `getAllocations()` before preparing** — gives user-friendly "not eligible" messages
4. **Handle each error code explicitly** — don't just catch-all with a generic message
5. **For bots: implement retry logic** — `API_ERROR` and `TRANSACTION_FAILED` may be transient

## React Error Handling Pattern

```tsx
const [error, setError] = useState<string | null>(null);

const handleMint = async () => {
  setError(null);
  try {
    const status = await product.getStatus();
    if (status !== 'active') {
      setError(`This drop is ${status}`);
      return;
    }

    const allocations = await product.getAllocations({
      recipientAddress: address,
    });
    if (!allocations.isEligible) {
      setError(allocations.reason || 'Not eligible to mint');
      return;
    }

    const prepared = await product.preparePurchase({
      userAddress: address,
      payload: { quantity: 1 },
      account,
    });

    const result = await product.purchase({
      account,
      preparedPurchase: prepared,
    });

    console.log('Success:', result.transactionReceipt.txHash);
  } catch (err) {
    if (err instanceof ClientSDKError) {
      setError(err.message);
    } else {
      setError('An unexpected error occurred');
    }
  }
};
```
