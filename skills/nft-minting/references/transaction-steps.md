# Transaction Steps

A purchase may require multiple on-chain transactions. The `PreparedPurchase.steps` array contains each step in order.

## When Multiple Steps Occur

- **ERC-20 priced products** — requires an approval transaction before the mint
- **Standard ETH-priced products** — typically a single mint step

## TransactionStep Interface

```typescript
interface TransactionStep {
  id: string;                    // e.g., 'approve-usdc', 'mint'
  name: string;                  // e.g., 'Approve USDC Spending'
  type: string;                  // 'approve' | 'mint'
  description: string;           // Human-readable description
  cost?: {                       // Cost for this specific step
    native?: Money;
    erc20s?: Money[];
  };
  transactionData: {
    contractAddress: string;
    value: bigint;               // Native token value
    transactionData: string;     // Encoded calldata
    gasEstimate: bigint;
    networkId: number;
  };
  execute: (
    account: IAccount,
    options?: { confirmations?: number }
  ) => Promise<Receipt>;
}
```

## Simple Execution (Recommended)

Let `product.purchase()` handle all steps sequentially:

```typescript
const result = await product.purchase({
  account,
  preparedPurchase: prepared,
});
```

This iterates through all steps, executing each one. If any step fails, it throws a `ClientSDKError` with `TRANSACTION_FAILED` and includes receipts from completed steps.

## Manual Step Execution

For UIs that need to show progress per step:

```typescript
const prepared = await product.preparePurchase({
  userAddress: address,
  payload: { quantity: 1 },
});

console.log(`${prepared.steps.length} step(s) required`);

for (let i = 0; i < prepared.steps.length; i++) {
  const step = prepared.steps[i];
  console.log(`Step ${i + 1}/${prepared.steps.length}: ${step.name}`);
  console.log(`  Type: ${step.type}`);
  console.log(`  Description: ${step.description}`);

  try {
    const receipt = await step.execute(account, { confirmations: 1 });
    console.log(`  ✅ TX: ${receipt.transactionReceipt.txHash}`);
  } catch (error) {
    console.error(`  ❌ Failed: ${error.message}`);
    throw error; // Don't continue if a step fails
  }
}
```

## React Step-by-Step UI Pattern

```tsx
const [currentStep, setCurrentStep] = useState(0);
const [stepStatuses, setStepStatuses] = useState<string[]>([]);

const handlePreparePurchase = async () => {
  const prepared = await product.preparePurchase({
    userAddress: address,
    payload: { quantity },
  });

  setPreparedPurchase(prepared);
  setStepStatuses(new Array(prepared.steps.length).fill('idle'));
  setCurrentStep(0);
};

const handleExecuteStep = async (stepIndex: number) => {
  const step = preparedPurchase.steps[stepIndex];

  setStepStatuses((prev) => {
    const next = [...prev];
    next[stepIndex] = 'executing';
    return next;
  });

  try {
    const account = createAccountViem({ walletClient });
    await step.execute(account);

    setStepStatuses((prev) => {
      const next = [...prev];
      next[stepIndex] = 'completed';
      return next;
    });

    if (stepIndex < preparedPurchase.steps.length - 1) {
      setCurrentStep(stepIndex + 1);
    }
  } catch (error) {
    setStepStatuses((prev) => {
      const next = [...prev];
      next[stepIndex] = 'failed';
      return next;
    });
  }
};
```

## Transaction Data (Read-Only)

Each step exposes `transactionData` for inspection without executing:

```typescript
for (const step of prepared.steps) {
  console.log(`Contract: ${step.transactionData.contractAddress}`);
  console.log(`Value: ${step.transactionData.value}`);
  console.log(`Gas estimate: ${step.transactionData.gasEstimate}`);
  console.log(`Network: ${step.transactionData.networkId}`);
}
```

## Receipt Object

```typescript
interface Receipt {
  transactionReceipt: {
    networkId: number;
    txHash: string;
    blockNumber?: number;
    gasUsed?: bigint;
  };
  order?: TokenOrder; // Present on mint steps for Edition products
}
```

## TokenOrder (Edition Products)

After a successful Edition mint, the receipt includes order details:

```typescript
interface TokenOrder {
  recipientAddress: string;
  total: Cost;
  items: TokenOrderItem[];
}

interface TokenOrderItem {
  quantity: number;
  token: Token;
  total: Cost;
}

interface Token {
  networkId: number;
  contract: Contract;
  tokenId: string;
  explorerUrl: object;
  media: Media;
}
```
