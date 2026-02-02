---
name: buy
description: Pay for x402-protected API endpoints with USDC. Use when calling APIs that return HTTP 402 Payment Required, integrating payments into agents, handling x402 payment requirements, building autonomous agents that pay for API access, or discovering paid services via Bazaar. Supports EVM (Base, Ethereum, Avalanche) and Solana networks.
---

# x402 Buy Skill

Enable AI agents to autonomously pay for HTTP resources using the x402 payment protocol.

## Overview

x402 is an open payment standard that enables programmatic payments over HTTP. When an agent requests a paid resource, the server responds with HTTP 402 (Payment Required) containing payment details. The agent signs a payment authorization and retries the request.

## Quick Start

### 1. Setup Wallet

Create a wallet using `npx add-wallet` and select wallet type (EVM or Solana).

Or use a private key directly:

```typescript
import { privateKeyToAccount } from "viem/accounts";
const signer = privateKeyToAccount(process.env.EVM_PRIVATE_KEY as `0x${string}`);
```

### 2. Install Packages

```bash
npm install @x402/fetch @x402/evm    # For EVM networks
npm install @x402/svm                 # For Solana (optional)
```

### 3. Make Paid Requests

```typescript
import { wrapFetchWithPayment, x402Client } from "@x402/fetch";
import { registerExactEvmScheme } from "@x402/evm/exact/client";
import { privateKeyToAccount } from "viem/accounts";

const signer = privateKeyToAccount(process.env.EVM_PRIVATE_KEY as `0x${string}`);
const client = new x402Client();
registerExactEvmScheme(client, { signer });

const fetchWithPayment = wrapFetchWithPayment(fetch, client);
const response = await fetchWithPayment("https://api.example.com/paid", { method: "GET" });
```

## Payment Flow

1. **Initial Request** - Make HTTP request to resource
2. **402 Response** - Server returns payment requirements
3. **Parse Requirements** - Extract from `PAYMENT-REQUIRED` header
4. **Create Payment** - Sign authorization with wallet
5. **Retry with Payment** - Include `PAYMENT-SIGNATURE` header
6. **Receive Resource** - Get response with `PAYMENT-RESPONSE` settlement details

## Network Support

| Network | CAIP-2 ID | Testnet |
|---------|-----------|---------|
| Base Sepolia | `eip155:84532` | Yes |
| Base Mainnet | `eip155:8453` | No |
| Avalanche Fuji | `eip155:43113` | Yes |
| Avalanche C-Chain | `eip155:43114` | No |
| Solana Devnet | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | Yes |
| Solana Mainnet | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | No |

## USDC Token Addresses

| Network | USDC Address |
|---------|-------------|
| Base Mainnet (`eip155:8453`) | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| Base Sepolia (`eip155:84532`) | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| Avalanche (`eip155:43114`) | `0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E` |
| Solana Mainnet | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |

## Multi-Network Client Setup

```typescript
import { x402Client } from "@x402/core/client";
import { registerExactEvmScheme } from "@x402/evm/exact/client";
import { registerExactSvmScheme } from "@x402/svm/exact/client";
import { privateKeyToAccount } from "viem/accounts";
import { createKeyPairSignerFromBytes } from "@solana/kit";
import { base58 } from "@scure/base";

const evmSigner = privateKeyToAccount(process.env.EVM_PRIVATE_KEY as `0x${string}`);
const svmSigner = await createKeyPairSignerFromBytes(
  base58.decode(process.env.SOLANA_PRIVATE_KEY!)
);

const client = new x402Client();
registerExactEvmScheme(client, { signer: evmSigner });
registerExactSvmScheme(client, { signer: svmSigner });
```

## Service Discovery (Bazaar)

The Bazaar helps agents find x402-enabled services.

### Discover Available Services

```typescript
import { HTTPFacilitatorClient } from "@x402/core/http";
import { withBazaar } from "@x402/extensions";

const facilitatorClient = new HTTPFacilitatorClient({
  url: "https://api.cdp.coinbase.com/platform/v2/x402"
});
const client = withBazaar(facilitatorClient);

const response = await client.extensions.discovery.listResources({ type: "http" });
```

### Filter and Call Services

```typescript
// Filter by price (under $0.10)
const usdcAsset = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913";
const maxPrice = 100000;

const affordableServices = response.items.filter(item =>
  item.accepts.find(req =>
    req.asset === usdcAsset &&
    Number(req.maxAmountRequired) < maxPrice
  )
);

// Call a discovered service
const selectedService = affordableServices[0];
const api = wrapAxiosWithPayment(axios.create(), client);
const result = await api.get(selectedService.resource);
```

## Manual 402 Handling

```typescript
import { x402Client } from "@x402/core/client";
import {
  decodePaymentRequiredHeader,
  encodePaymentSignatureHeader,
} from "@x402/core/http";

let response = await fetch(url);

if (response.status === 402) {
  const paymentRequired = decodePaymentRequiredHeader(
    response.headers.get("PAYMENT-REQUIRED")
  );
  const paymentPayload = await client.createPaymentPayload(paymentRequired);
  const paymentHeader = encodePaymentSignatureHeader(paymentPayload);

  response = await fetch(url, {
    headers: { "PAYMENT-SIGNATURE": paymentHeader }
  });
}
```

## Payment Amounts

| Amount (atomic) | USDC Value |
|-----------------|------------|
| `1000` | $0.001 (0.1 cents) |
| `10000` | $0.01 (1 cent) |
| `100000` | $0.10 (10 cents) |
| `1000000` | $1.00 |

## Setup Checklist

1. **Create wallet** - Run `npx add-wallet`
2. **Fund wallet** - Get testnet USDC from https://faucet.circle.com/
3. **Install SDK** - `npm install @x402/fetch @x402/evm`
4. **Register scheme** - `registerExactEvmScheme(client, { signer })`
5. **Wrap fetch** - `wrapFetchWithPayment(fetch, client)`
6. **Make requests** - SDK handles 402 → payment → retry automatically

## Key Packages

### TypeScript
- `@x402/fetch` - Fetch wrapper with auto payment
- `@x402/axios` - Axios interceptor with auto payment
- `@x402/core` - Core types and utilities
- `@x402/evm` - EVM network support
- `@x402/svm` - Solana network support

### Go
- `github.com/coinbase/x402/go` - Core client
- `github.com/coinbase/x402/go/http` - HTTP wrapper

## Protocol References

For detailed protocol schemas, see:
- [references/wallet-setup.md](references/wallet-setup.md)
- [references/v1-protocol.md](references/v1-protocol.md)
- [references/v2-protocol.md](references/v2-protocol.md)
