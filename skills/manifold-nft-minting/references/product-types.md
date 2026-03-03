# Product Types

The SDK supports two product types, with more planned.

## AppType Enum

```typescript
enum AppType {
  EDITION = 'edition',       // Fixed/open edition NFTs
  BLIND_MINT = 'blind-mint', // Mystery/gacha-style random NFTs
}
```

## AppId Enum (Internal IDs)

```typescript
enum AppId {
  EDITION = 2522713783,
  BLIND_MINT_1155 = 2526777015,
  BURN_REDEEM = 2520944311,
}
```

## Edition Product

Standard NFT editions — fixed or open supply.

**Created via:** [Manifold Studio Editions](https://help.manifold.xyz/en/articles/9387344-create-an-edition-open-or-limited)

**Features:**
- ERC-721 or ERC-1155 token standards
- Limited or unlimited supply
- Per-wallet mint limits
- Time-based sales windows (start/end dates)
- Allowlist support with merkle proofs
- ETH or ERC-20 pricing
- Redemption codes

**Type guard:**

```typescript
import { isEditionProduct } from '@manifoldxyz/client-sdk';

const product = await client.getProduct(instanceId);
if (isEditionProduct(product)) {
  // TypeScript knows this is EditionProduct
  const onchain = await product.fetchOnchainData();
  console.log(`Price: ${onchain.cost.formatted}`);
}
```

**Edition-specific onchain data:**
- `totalMax` — max supply (null = unlimited)
- `total` — total minted
- `walletMax` — per-wallet limit (null = unlimited)
- `cost` — price as Money object
- `startDate` / `endDate` — sale window
- `audienceType` — 'None' or 'Allowlist'
- `platformFee` / `merklePlatformFee` — Manifold fees
- `identical` (ERC-721) — whether all tokens share metadata
- `tokenId` (ERC-1155) — the token ID being minted

## Blind Mint Product

Mystery/gacha-style NFTs — buyers receive random items from a pool.

**Created via:** [Manifold Studio Serendipity](https://help.manifold.xyz/en/articles/9449681-serendipity)

**Features:**
- ERC-1155 only
- Tier-based probability system
- Token variations with different rarities
- Immediate or delayed reveals
- Pool-based inventory

**Type guard:**

```typescript
import { isBlindMintProduct } from '@manifoldxyz/client-sdk';

const product = await client.getProduct(instanceId);
if (isBlindMintProduct(product)) {
  // TypeScript knows this is BlindMintProduct
  const tiers = await product.getTierProbabilities();
  const variations = await product.getTokenVariations();
}
```

**BlindMint-specific methods:**
- `getTokenVariations()` — all possible NFTs in the pool with metadata and tier
- `getTierProbabilities()` — rarity tiers with drop rates
- `getClaimableTokens(walletAddress)` — what a wallet can claim

**BlindMint-specific onchain data:**
- `totalSupply` — max supply
- `totalMinted` — current minted count
- `cost` — price as Money object
- `tokenVariations` — number of unique token types
- `startingTokenId` — first token ID in the series
- `startDate` / `endDate` — sale window

## Checking Product Type After Fetch

Always use type guards before accessing type-specific methods:

```typescript
const product = await client.getProduct(instanceId);

if (isEditionProduct(product)) {
  // Edition-specific logic
  const rules = await product.getRules();
  console.log(`Max per wallet: ${rules.maxPerWallet}`);
} else if (isBlindMintProduct(product)) {
  // BlindMint-specific logic
  const tiers = await product.getTierProbabilities();
  console.log(`Tiers: ${tiers.length}`);
} else {
  throw new Error(`Unsupported product type: ${product.type}`);
}
```
