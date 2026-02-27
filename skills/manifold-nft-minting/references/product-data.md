# Product Data Methods

These methods are available on both Edition and BlindMint products.

## getStatus()

Returns the current sale status.

```typescript
const status = await product.getStatus();
// Returns: 'active' | 'upcoming' | 'sold-out' | 'ended'
```

| Status | Meaning |
|--------|---------|
| `active` | Sale is live, accepting purchases |
| `upcoming` | Start date hasn't been reached |
| `sold-out` | All supply has been minted |
| `ended` | End date has passed |

**Always check status before attempting purchases.**

## getAllocations(params)

Check how many tokens a wallet can mint.

```typescript
const allocations = await product.getAllocations({
  recipientAddress: '0x...',
});

if (!allocations.isEligible) {
  console.log('Cannot mint:', allocations.reason);
  return;
}
console.log('Can mint:', allocations.quantity); // null = unlimited
```

**Returns:**

```typescript
interface AllocationResponse {
  isEligible: boolean;    // Can this wallet purchase?
  quantity: number | null; // How many (null = unlimited)
  reason?: string;        // Why not eligible
}
```

**Possible `reason` values:**
- `"You are not on the allowlist"`
- `"You have used up all your allotted slots"`
- `"You have reached the maximum per wallet"`
- `"No mints available"`

## getInventory()

Get supply information.

```typescript
const inventory = await product.getInventory();
console.log(`Minted: ${inventory.totalPurchased} / ${inventory.totalSupply}`);
```

**Returns:**

```typescript
interface ProductInventory {
  totalSupply: number;     // -1 = unlimited
  totalPurchased: number;
}
```

## getRules()

Get product configuration rules.

```typescript
const rules = await product.getRules();
console.log(`Start: ${rules.startDate}`);
console.log(`End: ${rules.endDate}`);
console.log(`Audience: ${rules.audienceRestriction}`); // 'allowlist' | 'none'
console.log(`Max per wallet: ${rules.maxPerWallet}`);
```

**Returns:**

```typescript
interface ProductRule {
  startDate?: Date;                    // undefined = immediate
  endDate?: Date;                      // undefined = no end
  audienceRestriction: 'allowlist' | 'none';
  maxPerWallet?: number;               // undefined = unlimited
}
```

## getProvenance()

Get creator and contract info.

```typescript
const provenance = await product.getProvenance();
console.log(`Creator: ${provenance.creator.name} (@${provenance.creator.slug})`);
console.log(`Contract: ${provenance.contract.contractAddress}`);
console.log(`Network: ${provenance.networkId}`);
```

**Returns:**

```typescript
interface ProductProvenance {
  creator: {
    id: string;
    slug: string;
    address: string;
    name: string;
  };
  contract: Contract;
  networkId: number;
}

interface Contract {
  name: string;
  symbol: string;
  contractAddress: string;
  networkId: number;
  spec: 'erc721' | 'erc1155';
  explorer: { etherscanUrl: string };
}
```

## getMetadata()

Get product name and description.

```typescript
const metadata = await product.getMetadata();
console.log(`Name: ${metadata.name}`);
console.log(`Description: ${metadata.description}`);
```

## getPreviewMedia()

Get product media URLs.

```typescript
const media = await product.getPreviewMedia();
console.log(`Image: ${media?.image}`);
console.log(`Preview: ${media?.imagePreview}`);
console.log(`Animation: ${media?.animation}`);
```

**Returns:**

```typescript
interface Media {
  image?: string;
  imagePreview?: string;
  animation?: string;
  animationPreview?: string;
}
```

## fetchOnchainData()

Fetches on-chain data (price, supply, dates) directly from the blockchain. Results are cached — pass `force: true` to refresh.

```typescript
const onchain = await product.fetchOnchainData();
console.log(`Price: ${onchain.cost.formatted}`);
```

See `references/product-types.md` for type-specific onchain data fields.

## Accessing Off-Chain Data (InstanceData)

Product data from the API is available at `product.data`:

```typescript
// Public data
const publicData = product.data.publicData;
console.log(`Title: ${publicData.name || publicData.title}`);
console.log(`Network: ${publicData.network}`);

// Creator info
const creator = product.data.creator;
console.log(`Creator: ${creator.name} (${creator.slug})`);

// Preview data
const preview = product.previewData;
console.log(`Thumbnail: ${preview.thumbnail}`);
console.log(`Description: ${preview.description}`);
```

## Display Example

```typescript
const product = await client.getProduct(instanceId);

const metadata = await product.getMetadata();
const media = await product.getPreviewMedia();
const inventory = await product.getInventory();
const rules = await product.getRules();
const onchain = await product.fetchOnchainData();

console.log(`${metadata.name}`);
console.log(`${metadata.description}`);
console.log(`Image: ${media?.image}`);
console.log(`Price: ${onchain.cost.formatted}`);
console.log(`Minted: ${inventory.totalPurchased}/${inventory.totalSupply === -1 ? '∞' : inventory.totalSupply}`);
console.log(`Audience: ${rules.audienceRestriction}`);
```
