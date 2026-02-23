---
name: nft-minting
description: Build NFT minting experiences with Manifold's client-sdk. Guides agents through campaign setup, custom minting websites (React/Next.js), minting bots (Node.js), and SDK integration into existing projects. Supports Edition and Blind Mint products across Ethereum, Base, Optimism, Shape, Sepolia, and ApeChain. Use when building minting pages, mint bots, integrating Manifold NFT products, or helping users set up Manifold campaigns. NOT for: deploying smart contracts, managing Manifold Studio settings, or non-minting blockchain operations.
---

# Manifold NFT Minting Skill

Build custom NFT minting experiences using [`@manifoldxyz/client-sdk`](https://github.com/manifoldxyz/client-sdk).

## References

Load lazily — only when the current step needs them. **Never load all at once.**

| Reference | When to Load |
|-----------|-------------|
| `getting-started.md` | First SDK setup in any project |
| `product-types.md` | Working with Edition vs BlindMint products |
| `purchase-flow.md` | Implementing preparePurchase → purchase |
| `transaction-steps.md` | Multi-step transactions, ERC-20 approvals |
| `adapters.md` | Setting up wallet + provider adapters (ethers5/viem/wagmi) |
| `product-data.md` | Querying status, allocations, inventory, rules |
| `error-handling.md` | Error codes, pitfalls — **always load before writing purchase code** |
| `networks.md` | Multi-chain setup or non-mainnet deployments |
| `react-minting-app.md` | Building a React/Next.js minting page |
| `minting-bot.md` | Building a headless minting bot |
| `studio-setup-guide.md` | User needs to create a Manifold campaign first |
| `full-docs.md` | Fallback only — 128KB complete SDK docs with TOC for grep |

## Workflow

Follow in order. Ask questions — don't assume.

### Step 1: Check if they have a Manifold campaign

Ask: **"Do you already have a Manifold product/campaign deployed with an instance ID?"**

- **No** → Read `references/studio-setup-guide.md`. Guide them through product type selection and Studio setup. Return when they have an instance ID.
- **Yes** → Continue.

### Step 2: Determine project context

Ask: **"Do you have an existing project to add minting to, or building from scratch?"**

**Existing project:**
1. Ask their framework and wallet library (wagmi, viem, ethers v5)
2. Read `references/getting-started.md` + `references/adapters.md`
3. Match adapter setup to their stack — integrate, don't scaffold

**From scratch:**
1. Ask what they're building:
   - **Web minting page** → Read `references/react-minting-app.md`
   - **Server-side bot** → Read `references/minting-bot.md`
   - **Data query script** → Read `references/getting-started.md` only

### Step 3: Implement product interaction

1. Read `references/product-types.md` — use type guards (`isEditionProduct`, `isBlindMintProduct`)
2. Read `references/product-data.md` — status, allocations, metadata display
3. Read `references/purchase-flow.md` — two-step prepare → purchase
4. If ERC-20 pricing → also read `references/transaction-steps.md`

### Step 4: Error handling

**Always before finalizing:**

1. Read `references/error-handling.md` — ClientSDKError handling + common pitfalls
2. If multi-chain → read `references/networks.md`

## Rules

- **Never fabricate SDK method signatures or field names.** Verify against references.
- **Always use type guards** before accessing product-specific methods.
- **Always check `getStatus()`** before purchases.
- **Use `preparePurchase` → `purchase`** for simple flows. Manual `step.execute()` only for granular UI control.
- **Never hardcode chain IDs** — use configuration.
- **Never commit private keys** — use environment variables.
- Verify code against official docs at [docs.manifold.xyz/client-sdk](https://docs.manifold.xyz/client-sdk/).
- For issues beyond SDK scope → [Manifold Help](https://help.manifold.xyz/) or [Forum](https://forum.manifold.xyz).
