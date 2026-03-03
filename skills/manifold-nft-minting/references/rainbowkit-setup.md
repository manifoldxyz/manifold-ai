# RainbowKit Setup Guide

## Installation

> **CRITICAL: Always install `wagmi@^2.9.0` — NEVER run `npm install wagmi` without the version pin.** Running `npm install wagmi` or `npm install wagmi@latest` installs wagmi 3.x, which is incompatible with RainbowKit.

```bash
npm install @rainbow-me/rainbowkit wagmi@^2.9.0 viem@2.x @tanstack/react-query
```

## Wagmi Config

Create a config file (e.g., `wagmi.ts`):

```typescript
import { getDefaultConfig } from '@rainbow-me/rainbowkit';
import { mainnet, base, optimism, sepolia } from 'wagmi/chains';

export const config = getDefaultConfig({
  appName: 'My Minting App',
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  chains: [mainnet, base, optimism, sepolia],
  ssr: true, // Required for Next.js
});
```

Get a free WalletConnect `projectId` at [cloud.walletconnect.com](https://cloud.walletconnect.com/).

### Production RPC Transports

For production apps, override the default public RPCs with dedicated providers:

```typescript
import { getDefaultConfig } from '@rainbow-me/rainbowkit';
import { http } from 'wagmi';
import { mainnet, base } from 'wagmi/chains';

export const config = getDefaultConfig({
  appName: 'My Minting App',
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  chains: [mainnet, base],
  transports: {
    [mainnet.id]: http(process.env.NEXT_PUBLIC_RPC_URL_MAINNET),
    [base.id]: http(process.env.NEXT_PUBLIC_RPC_URL_BASE),
  },
  ssr: true,
});
```

## Provider Wrapper

Wrap your app with the required providers in your root layout or `_app.tsx`:

```tsx
'use client';

import { RainbowKitProvider } from '@rainbow-me/rainbowkit';
import { WagmiProvider } from 'wagmi';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { config } from './wagmi';

import '@rainbow-me/rainbowkit/styles.css';

const queryClient = new QueryClient();

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider>
          {children}
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

**Provider nesting order matters:** `WagmiProvider` > `QueryClientProvider` > `RainbowKitProvider`.

## ConnectButton

```tsx
import { ConnectButton } from '@rainbow-me/rainbowkit';

export function Header() {
  return (
    <nav>
      <ConnectButton />
    </nav>
  );
}
```

### ConnectButton Props

| Prop | Type | Description |
|------|------|-------------|
| `label` | `string` | Custom text for the connect button (default: "Connect Wallet") |
| `accountStatus` | `'full' \| 'avatar' \| 'address'` | What to show when connected |
| `chainStatus` | `'full' \| 'icon' \| 'name' \| 'none'` | How to display chain info |
| `showBalance` | `boolean \| { smallScreen: boolean; largeScreen: boolean }` | Whether to show balance |

```tsx
<ConnectButton
  label="Connect to Mint"
  accountStatus="address"
  chainStatus="icon"
  showBalance={false}
/>
```

### Custom Connect Button

Use `ConnectButton.Custom` for full UI control:

```tsx
<ConnectButton.Custom>
  {({ account, chain, openConnectModal, openChainModal, mounted }) => {
    const connected = mounted && account && chain;
    return (
      <div>
        {!connected ? (
          <button onClick={openConnectModal}>Connect Wallet</button>
        ) : (
          <button onClick={openChainModal}>
            {account.displayName} ({chain.name})
          </button>
        )}
      </div>
    );
  }}
</ConnectButton.Custom>
```

## Next.js Notes

- If using Next.js with Turbopack, run `next dev --webpack` instead of `next dev` to avoid RainbowKit build issues.
- Mark provider components as `'use client'` in App Router.
- Set `ssr: true` in the wagmi config for server-side rendering compatibility.
