# Setting Up a Manifold Campaign

Before using the SDK, you need a deployed product on [Manifold Studio](https://studio.manifold.xyz). This guide helps you pick the right product type and get started.

## Quick Decision Tree

**What are you creating?**

1. **A single unique artwork** → [1 of 1](#1-of-1)
2. **Multiple copies of the same artwork** → [Edition](#editions)
3. **A mystery/gacha drop with random items** → [Blind Mint (Serendipity)](#blind-mint-serendipity)
4. **A token exchange (burn X to get Y)** → [Burn Redeem](#burn-redeem)
5. **Multiple unique artworks at once** → [Batch Mint](#batch-mint)
6. **A physical item with NFT** → [Get Physical](#get-physical)
7. **Selling/auctioning a minted NFT** → [Gallery Listing](#gallery-listing)

## Product Types

### 1 of 1

Mint a single unique artwork and sell it on any marketplace.

- **Help docs:** https://help.manifold.xyz/en/articles/9387342-create-a-1-of-1
- **SDK support:** Not directly (1/1s are minted by the creator, sold on marketplaces)

### Editions

Create a page for collectors to mint copies of your artwork. Most common for SDK integration.

- **Help docs:** https://help.manifold.xyz/en/articles/9387344-create-an-edition-open-or-limited
- **SDK support:** ✅ Full support via `EditionProduct`
- **Features:**
  - Open (unlimited) or limited supply
  - ERC-721 or ERC-1155
  - Per-wallet mint limits
  - Time-based sales (start/end dates)
  - Allowlist support
  - ETH or ERC-20 pricing
  - Claim codes

### Blind Mint (Serendipity)

Create a mystery drop where collectors receive random NFTs with different rarities.

- **Help docs:** https://help.manifold.xyz/en/articles/9449681-serendipity
- **SDK support:** ✅ Full support via `BlindMintProduct`
- **Features:**
  - ERC-1155
  - Tiered rarity system
  - Configurable probabilities
  - Multiple token variations
  - Reveal mechanics

### Burn Redeem

Exchange existing tokens for new ones (burn old NFT → receive new NFT).

- **Help docs:** https://help.manifold.xyz/en/articles/9387352-create-a-burn-redeem-campaign
- **SDK support:** ⚠️ AppId defined but not yet implemented in SDK

### Batch Mint

Mint multiple unique 1/1 tokens at once, each with different media and metadata.

- **Help docs:** https://help.manifold.xyz/en/articles/9387360-create-a-batch-mint-of-1-1-tokens
- **SDK support:** Not directly (creator-side minting)

### Get Physical

Sell physical items with NFT + ETH payments.

- **Help docs:** https://help.manifold.xyz/en/articles/9450556-get-physical
- **SDK support:** Not directly

### Gallery Listing

List a minted NFT for sale or auction via Manifold.

- **Help docs:** https://help.manifold.xyz/en/articles/9387366-create-a-gallery-listing
- **SDK support:** Not directly

## Creating a Product (Step by Step)

1. Go to [studio.manifold.xyz](https://studio.manifold.xyz) and sign in
2. Click the **Create+** menu
3. Choose your product type (Edition or Serendipity for SDK use)
4. Configure:
   - **Asset** — upload your artwork (image, video, audio, etc.)
   - **Network** — choose which blockchain (Ethereum, Base, Optimism, Shape, etc.)
   - **Supply** — limited or unlimited
   - **Price** — free, ETH, or ERC-20 token
   - **Allowlist** — optional list of approved wallets
   - **Dates** — optional start/end times
5. **Publish** the product
6. **Copy the instance ID** from the URL — you'll need this for the SDK

## Finding Your Instance ID

After publishing, the product URL looks like:

```
https://studio.manifold.xyz/apps/[APP_ID]/instance/[INSTANCE_ID]
```

Or on the collector page:

```
https://manifold.xyz/@creator/id/[INSTANCE_ID]
```

The instance ID is the numeric value (e.g., `4150231280`). You can use either the ID or the full URL with the SDK.

## Testing on Sepolia

For development, create products on the **Sepolia testnet**:

1. In Studio, select **Sepolia** as the network when creating
2. Get testnet ETH from a [faucet](https://sepoliafaucet.com/)
3. Test your minting flow without spending real ETH
4. Switch to mainnet when ready for production

## Additional Resources

- **Full Create+ Menu docs:** https://help.manifold.xyz/en/collections/9493376-your-create-menu
- **Edition details:** https://help.manifold.xyz/en/collections/9493378-editions-formerly-claims
- **Community Forum:** https://forum.manifold.xyz
- **Discord:** https://discord.gg/cGgTZZGTeV
