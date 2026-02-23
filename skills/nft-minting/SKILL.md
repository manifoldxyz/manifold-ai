---
name: nft-minting
description: Build NFT minting experiences with Manifold's client-sdk. Guides agents through campaign setup, custom minting websites (React/Next.js), minting bots (Node.js), and SDK integration into existing projects. Supports Edition and Blind Mint products across Ethereum, Base, Optimism, Shape, Sepolia, and ApeChain. Use when building minting pages, mint bots, integrating Manifold NFT products, or helping users set up Manifold campaigns.
---

# Manifold NFT Minting Skill

Build custom NFT minting experiences using [`@manifoldxyz/client-sdk`](https://github.com/manifoldxyz/client-sdk).

## Reference Files

This skill includes focused reference files in `references/`. Load them lazily — only when the current step needs them. **Never load all references at once.**

| Reference | When to Load |
|-----------|-------------|
| `getting-started.md` | First SDK setup in any project |
| `product-types.md` | Identifying or working with Edition vs BlindMint |
| `purchase-flow.md` | Implementing preparePurchase → purchase |
| `transaction-steps.md` | Multi-step transactions, ERC-20 approvals, step.execute() |
| `wallet-adapters.md` | Setting up account adapters (ethers5/viem/wagmi) |
| `public-providers.md` | Setting up public providers for blockchain reads |
| `product-data.md` | Querying status, allocations, inventory, rules, provenance |
| `error-handling.md` | Writing error handling for purchases (**always load before writing purchase code**) |
| `networks.md` | Multi-chain setup or non-mainnet deployments |
| `react-minting-app.md` | Building a React/Next.js minting page |
| `minting-bot.md` | Building a headless minting bot |
| `studio-setup-guide.md` | User needs to create a Manifold campaign first |
| `common-pitfalls.md` | **Always load before finalizing any implementation** |
| `full-docs.md` | Fallback — only if the task doesn't fit specific references |

## Workflow

Follow these steps in order. Ask questions — don't assume.

### Step 1: Check if they have a Manifold campaign

Ask: **"Do you already have a Manifold product/campaign deployed with an instance ID?"**

- **NO** → Read `references/studio-setup-guide.md`. Guide them through picking a product type and setting up via [Manifold Studio](https://studio.manifold.xyz). Come back when they have an instance ID.
- **YES** → Continue to Step 2.

### Step 2: Determine project context

Ask: **"Do you have an existing project you want to add minting to, or are you building from scratch?"**

#### Existing Project

1. Ask what framework/stack they're using (React, Next.js, Node.js, vanilla JS, etc.)
2. Ask what wallet library they use (wagmi, viem, ethers v5, or none yet)
3. Read `references/getting-started.md` for SDK installation
4. Read the matching adapter reference:
   - Using wagmi/React → `references/wallet-adapters.md` + `references/public-providers.md`
   - Using viem → `references/wallet-adapters.md` + `references/public-providers.md`
   - Using ethers v5 → `references/wallet-adapters.md` + `references/public-providers.md`
5. Integrate the SDK into their existing codebase — don't scaffold a new project

#### From Scratch

1. Ask: **"What are you building?"**
   - **Custom minting website** → Read `references/react-minting-app.md`, scaffold Next.js + RainbowKit + wagmi
   - **Minting bot / server script** → Read `references/minting-bot.md`, scaffold Node.js + viem or ethers5
   - **Just querying data** → Read `references/getting-started.md`, minimal script with public provider only

### Step 3: Implement the product interaction

Based on their product type:

1. Read `references/product-types.md` — use type guards (`isEditionProduct`, `isBlindMintProduct`)
2. Read `references/product-data.md` — fetch status, check allocations, display metadata
3. Read `references/purchase-flow.md` — implement the two-step prepare → purchase flow
4. If the product uses ERC-20 pricing → also read `references/transaction-steps.md`

### Step 4: Error handling and finalization

**Always before finalizing:**

1. Read `references/error-handling.md` — implement proper ClientSDKError handling
2. Read `references/common-pitfalls.md` — verify no LLM-specific mistakes
3. If multi-chain → read `references/networks.md`

## Key Rules

- **Never fabricate SDK method signatures or field names.** Always verify against the reference files.
- **Always use type guards** (`isEditionProduct`, `isBlindMintProduct`) before accessing product-specific methods.
- **Always check `getStatus()`** before attempting purchases.
- **Always handle errors** with proper `ClientSDKError` code checks.
- **Never hardcode chain IDs** — use configuration.
- **Never commit private keys** — use environment variables.
- **Use `preparePurchase` → `purchase` for simple flows.** Only use manual `step.execute()` when you need granular control (e.g., showing approval + mint as separate UI steps).
- If using example code, ensure it's from the official `@manifoldxyz/client-sdk` package or documentation at [docs.manifold.xyz/client-sdk](https://docs.manifold.xyz/client-sdk/).
- For issues beyond SDK scope, direct users to [Manifold Help](https://help.manifold.xyz/) or [Community Forum](https://forum.manifold.xyz).
