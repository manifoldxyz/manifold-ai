# manifold-ai

AI agent skills for building with [Manifold](https://manifold.xyz) — the marketplace of ideas.

## Available Skills

### nft-minting

Build custom NFT minting experiences using the [`@manifoldxyz/client-sdk`](https://github.com/manifoldxyz/client-sdk). Covers:

- **Campaign setup guidance** — pick the right Manifold product type and deploy via [Studio](https://studio.manifold.xyz/)
- **Custom minting websites** — React/Next.js apps with wallet connect and mint flows
- **Minting bots** — headless server-side scripts for automated minting
- **SDK integration** — add minting to existing projects with any wallet library (ethers v5, viem, wagmi)

Supports all Manifold chains: Ethereum, Base, Optimism, Shape, and Sepolia(testnet).

## Installation

```bash
npx skills add manifoldxyz/manifold-ai
```

Install a specific skill:

```bash
npx skills add manifoldxyz/manifold-ai --skill nft-minting
```

Install for a specific agent:

```bash
npx skills add manifoldxyz/manifold-ai -a claude-code
npx skills add manifoldxyz/manifold-ai -a cursor
npx skills add manifoldxyz/manifold-ai -a openclaw
```

## Resources

- [Manifold Client SDK Docs](https://docs.manifold.xyz/client-sdk/)
- [Manifold Studio](https://studio.manifold.xyz/)
- [Manifold Help Center](https://help.manifold.xyz/)
- [SDK GitHub](https://github.com/manifoldxyz/client-sdk)
- [Community Forum](https://forum.manifold.xyz)

## License

MIT
