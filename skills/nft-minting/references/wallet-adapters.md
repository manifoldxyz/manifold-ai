# Wallet Adapters (Account)

Account adapters wrap wallet libraries into the SDK's unified `IAccount` interface for signing and sending transactions.

## Available Adapters

| Adapter | Import | Use With |
|---------|--------|----------|
| `createAccountViem` | `@manifoldxyz/client-sdk/adapters` | viem `WalletClient` |
| `createAccountEthers5` | `@manifoldxyz/client-sdk/adapters` | ethers v5 `Wallet` or `Signer` |

Wagmi doesn't have its own account adapter — use `createAccountViem` with wagmi's `useWalletClient()`.

## Viem Adapter

### Browser (with wagmi + RainbowKit)

```typescript
import { createAccountViem } from '@manifoldxyz/client-sdk';
import { useWalletClient } from 'wagmi';

const { data: walletClient } = useWalletClient();
const account = createAccountViem({ walletClient });
```

### Server-side (with private key)

```typescript
import { createAccountViem } from '@manifoldxyz/client-sdk';
import { createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const wallet = privateKeyToAccount('0xYOUR_PRIVATE_KEY' as `0x${string}`);
const walletClient = createWalletClient({
  account: wallet,
  chain: base,
  transport: http('YOUR_RPC_URL'),
});
const account = createAccountViem({ walletClient });
```

## Ethers v5 Adapter

### Browser (with Web3Provider)

```typescript
import { createAccountEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const provider = new ethers.providers.Web3Provider(window.ethereum);
const signer = provider.getSigner();
const account = createAccountEthers5({ wallet: signer });
```

### Server-side (with private key)

```typescript
import { createAccountEthers5 } from '@manifoldxyz/client-sdk';
import { ethers } from 'ethers';

const provider = new ethers.providers.JsonRpcProvider('YOUR_RPC_URL');
const wallet = new ethers.Wallet('YOUR_PRIVATE_KEY', provider);
const account = createAccountEthers5({ wallet });
```

## IAccount Interface

All adapters implement this interface:

```typescript
interface IAccount {
  readonly adapterType: 'ethers5' | 'ethers6' | 'viem' | 'wagmi';
  getAddress(): Promise<string>;
  sendTransaction(request: UniversalTransactionRequest): Promise<string>;
  sendTransactionWithConfirmation(
    request: UniversalTransactionRequest,
    options?: { confirmations?: number }
  ): Promise<UniversalTransactionResponse>;
  getBalance(networkId: number, tokenAddress?: string): Promise<Money>;
  switchNetwork(chainId: number): Promise<void>;
  signMessage(message: string): Promise<string>;
  signTypedData?(typedData: TypedDataPayload): Promise<string>;
  sendCalls(method: string, params?: unknown[]): Promise<unknown>;
}
```

## Choosing an Adapter

| Scenario | Adapter | Why |
|----------|---------|-----|
| React app with wagmi/RainbowKit | `createAccountViem` + `useWalletClient()` | Native wagmi integration |
| React app with custom viem setup | `createAccountViem` | Direct viem support |
| Server-side bot (modern) | `createAccountViem` + `privateKeyToAccount` | Lightweight, modern |
| Server-side bot (legacy) | `createAccountEthers5` + `ethers.Wallet` | If already using ethers v5 |
| Existing ethers v5 codebase | `createAccountEthers5` | Drop-in compatibility |
