# manifold-ai

AI agent skills for building with [Manifold](https://manifold.xyz) — the marketplace of ideas.

This repository provides installable skill packs that teach AI coding agents (Claude Code, Cursor, OpenClaw, etc.) how to build with Manifold's SDKs and tools. Skills contain structured workflows, reference docs, and guardrails so agents produce correct, production-ready code.

## Available Skills

### manifold-nft-minting

Build custom NFT minting experiences using [`@manifoldxyz/client-sdk`](https://github.com/manifoldxyz/client-sdk). Covers:

- **Campaign setup guidance** — pick the right Manifold product type and deploy via [Studio](https://studio.manifold.xyz/)
- **Custom minting websites** — React/Next.js apps with wallet connect and mint flows
- **Minting bots** — headless server-side scripts for automated minting
- **SDK integration** — add minting to existing projects with any wallet library (ethers v5, viem, wagmi)

Supports all Manifold chains: Ethereum, Base, Optimism, Shape, ApeChain, and Sepolia (testnet).

## Installation

```bash
npx skills add manifoldxyz/manifold-ai
```

Install a specific skill:

```bash
npx skills add manifoldxyz/manifold-ai --skill manifold-nft-minting
```

Install for a specific agent:

```bash
npx skills add manifoldxyz/manifold-ai -a claude-code
npx skills add manifoldxyz/manifold-ai -a cursor
npx skills add manifoldxyz/manifold-ai -a openclaw
```

## Architecture

```
manifold-ai/
├── README.md
├── LICENSE                                    # MIT
└── skills/
    └── manifold-nft-minting/
        ├── SKILL.md                           # Skill definition (workflow, rules, triggers)
        └── references/                        # Lazily-loaded reference docs
            ├── getting-started.md             # First SDK setup in any project
            ├── product-types.md               # Edition vs BlindMint products
            ├── purchase-flow.md               # preparePurchase → purchase two-step flow
            ├── transaction-steps.md           # Multi-step transactions, ERC-20 approvals
            ├── adapters.md                    # Wallet + provider adapters (ethers5/viem/wagmi)
            ├── product-data.md                # Querying status, allocations, inventory, rules
            ├── error-handling.md              # Error codes, pitfalls, ClientSDKError
            ├── networks.md                    # Multi-chain setup, chain configuration
            ├── react-minting-app.md           # Building a React/Next.js minting page
            ├── rainbowkit-setup.md            # RainbowKit install, wagmi config, providers
            ├── rpc-setup-guide.md             # RPC provider setup (Alchemy/Infura/QuickNode)
            ├── minting-bot.md                 # Headless minting bot (Node.js)
            ├── studio-setup-guide.md          # Creating a Manifold campaign in Studio
            └── full-docs.md                   # Complete SDK docs (~128KB) for grep fallback
```

### Skill Format

Each skill lives in `skills/<skill-name>/` and contains:

| File | Purpose |
|------|---------|
| `SKILL.md` | YAML frontmatter (name, description, triggers) + markdown body with workflow steps, rules, and reference load instructions |
| `references/*.md` | Supporting docs loaded lazily by agents only when needed for the current step |

The `SKILL.md` workflow guides agents through an interactive process: asking the user questions, loading the right references at the right time, and enforcing rules (e.g., version pins, security practices).

## External Dependencies

### Internal Packages

| Package | Purpose | Repo |
|---------|---------|------|
| `@manifoldxyz/client-sdk` | Core SDK for NFT minting — product queries, purchases, wallet adapters | [client-sdk](https://github.com/manifoldxyz/client-sdk) |

### Third-Party Services & Libraries

| Service/Library | Purpose | Used In |
|-----------------|---------|---------|
| [Manifold Studio](https://studio.manifold.xyz/) | Campaign/product deployment dashboard | Campaign setup workflow |
| [RainbowKit](https://www.rainbowkit.com/) | Wallet connection UI for React apps | Web minting pages |
| [wagmi](https://wagmi.sh/) (v2.x) | React hooks for Ethereum — **must pin `^2.9.0` with RainbowKit** | Web minting pages |
| [viem](https://viem.sh/) | TypeScript Ethereum interface (preferred over ethers) | All SDK interactions |
| [ethers v5](https://docs.ethers.org/v5/) | Alternative Ethereum library (legacy support) | Existing ethers projects |
| [Alchemy](https://www.alchemy.com/) / [Infura](https://www.infura.io/) / [QuickNode](https://www.quicknode.com/) | RPC node providers | Chain connectivity |

### Supported Chains

| Chain | Network | Notes |
|-------|---------|-------|
| Ethereum | Mainnet | Primary chain |
| Base | Mainnet | L2 |
| Optimism | Mainnet | L2 |
| Shape | Mainnet | L2 |
| ApeChain | Mainnet | |
| Sepolia | Testnet | For development/testing |

## Contributing

To add a new skill:

1. Create a directory under `skills/<skill-name>/`
2. Write a `SKILL.md` with YAML frontmatter (`name`, `description`) and a markdown body containing:
   - **References table** — list reference docs with load conditions
   - **Workflow** — numbered steps the agent follows interactively
   - **Rules** — guardrails and constraints
3. Add reference docs in `skills/<skill-name>/references/` — one file per topic, loaded lazily
4. Keep `full-docs.md` as a grep-able fallback for edge cases not covered by focused references

### Skill Design Principles

- **Lazy loading** — agents load references only when the current step needs them, never all at once
- **Interactive workflow** — skills ask the user questions rather than assuming context
- **Guardrails over freedom** — explicit rules prevent common agent mistakes (e.g., wagmi version pinning)
- **Verify against official docs** — skills reference canonical sources, not fabricated API signatures

## Resources

- [Manifold Client SDK Docs](https://docs.manifold.xyz/client-sdk/)
- [Manifold Studio](https://studio.manifold.xyz/)
- [Manifold Help Center](https://help.manifold.xyz/)
- [SDK GitHub](https://github.com/manifoldxyz/client-sdk)
- [Community Forum](https://forum.manifold.xyz)

## License

MIT
