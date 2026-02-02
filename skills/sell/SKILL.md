---
name: sell
description: Accept USDC payments for API endpoints using x402 protocol. Use when creating paid API endpoints, monetizing services with micropayments, building AI agents that sell data or services, or setting up payment middleware for Express, Hono, Next.js, FastAPI, Flask, or Go servers. Supports EVM (Base, Ethereum, Avalanche) and Solana via CDP Facilitator.
---

# x402 Sell Skill

Enable AI agents to sell resources and accept USDC payments for API endpoints using x402.

## Overview

x402 allows servers to payment-gate endpoints. When clients request a protected resource without payment, the server returns HTTP 402 with payment requirements. The server uses a **Facilitator** (like Coinbase CDP) to verify and settle payments on-chain.

## Quick Start

### 1. Setup Receiving Wallet

Create a wallet to receive payments.

**Non-interactive CLI commands (for AI agents):**

```bash
# EVM wallet (Base, Ethereum, Avalanche) - pipes "1" to select EOA
echo "1" | npx add-wallet evm

# Solana wallet
npx add-wallet sol

# Top up with testnet USDC (for testing payments)
npx add-wallet topup testnet
```

This creates a `.env` file with `WALLET_ADDRESS` and `WALLET_PRIVATE_KEY`.

**Use the address in your code:**

```typescript
// Read from .env or use directly
const payTo = process.env.WALLET_ADDRESS; // From .env file
// or
const payTo = "0xYourWalletAddress"; // EVM address
// or
const payTo = "YourSolanaAddress";   // Solana address
```

### 2. Install Packages

```bash
# Express
npm install @x402/express @x402/core @x402/evm

# Next.js
npm install @x402/next @x402/core @x402/evm

# Hono
npm install @x402/hono @x402/core @x402/evm

# For Solana support:
npm install @x402/svm
```

### 3. Add Payment Middleware (Express)

```typescript
import express from "express";
import { paymentMiddleware } from "@x402/express";
import { x402ResourceServer, HTTPFacilitatorClient } from "@x402/core/server";
import { registerExactEvmScheme } from "@x402/evm/exact/server";

const app = express();
const payTo = "0xYourAddress";

const facilitatorClient = new HTTPFacilitatorClient({
  url: "https://api.cdp.coinbase.com/platform/v2/x402"  // Mainnet (default)
  // url: "https://x402.org/facilitator"  // Testnet
});

const server = new x402ResourceServer(facilitatorClient);
registerExactEvmScheme(server);

app.use(
  paymentMiddleware(
    {
      "GET /weather": {
        accepts: [
          {
            scheme: "exact",
            price: "$0.001",
            network: "eip155:8453",  // Base Mainnet (use eip155:84532 for testnet)
            payTo,
          },
        ],
        description: "Get weather data",
        mimeType: "application/json",
      },
    },
    server,
  ),
);

app.get("/weather", (req, res) => {
  res.json({ weather: "sunny", temperature: 70 });
});

app.listen(4021);
```

## Facilitator URLs

**Default: Mainnet (CDP)** - Use for production payments.

| Environment | URL |
|-------------|-----|
| Mainnet (CDP) | `https://api.cdp.coinbase.com/platform/v2/x402` (default) |
| Testnet | `https://x402.org/facilitator` |

## Price Format

The `price` field uses dollar notation. SDK converts to atomic units automatically.

| Price | Meaning | Atomic Units |
|-------|---------|--------------|
| `"$0.001"` | 0.1 cents | 1000 |
| `"$0.01"` | 1 cent | 10000 |
| `"$0.10"` | 10 cents | 100000 |
| `"$1.00"` | 1 dollar | 1000000 |

## Network Identifiers (CAIP-2)

**Default: Base Mainnet (`eip155:8453`)** - Use for production payments.

| Network | CAIP-2 ID | Environment |
|---------|-----------|-------------|
| Base Mainnet | `eip155:8453` | Production (default) |
| Avalanche C-Chain | `eip155:43114` | Production |
| Solana Mainnet | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | Production |
| Base Sepolia | `eip155:84532` | Testnet |
| Avalanche Fuji | `eip155:43113` | Testnet |
| Solana Devnet | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | Testnet |

## USDC Token Addresses

| Network | USDC Contract Address |
|---------|----------------------|
| Base Mainnet (`eip155:8453`) | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| Base Sepolia (`eip155:84532`) | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| Avalanche (`eip155:43114`) | `0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E` |
| Solana Mainnet | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |

## Framework Examples

### Next.js

```typescript
// middleware.ts
import { paymentProxy } from "@x402/next";
import { x402ResourceServer, HTTPFacilitatorClient } from "@x402/core/server";
import { registerExactEvmScheme } from "@x402/evm/exact/server";

const facilitatorClient = new HTTPFacilitatorClient({
  url: "https://api.cdp.coinbase.com/platform/v2/x402"  // Mainnet
});
const server = new x402ResourceServer(facilitatorClient);
registerExactEvmScheme(server);

export const middleware = paymentProxy(
  {
    "/api/protected": {
      accepts: [{ scheme: "exact", price: "$0.01", network: "eip155:8453", payTo: "0xYour" }],
      description: "Protected content",
    },
  },
  server,
);

export const config = { matcher: ["/api/protected/:path*"] };
```

### Hono

```typescript
import { Hono } from "hono";
import { paymentMiddleware } from "@x402/hono";
import { x402ResourceServer, HTTPFacilitatorClient } from "@x402/core/server";
import { registerExactEvmScheme } from "@x402/evm/exact/server";

const app = new Hono();
const facilitatorClient = new HTTPFacilitatorClient({ url: "https://api.cdp.coinbase.com/platform/v2/x402" });
const server = new x402ResourceServer(facilitatorClient);
registerExactEvmScheme(server);

app.use(
  paymentMiddleware(
    {
      "/paid": {
        accepts: [{ scheme: "exact", price: "$0.10", network: "eip155:8453", payTo: "0xYour" }],
      },
    },
    server,
  ),
);

app.get("/paid", (c) => c.json({ message: "Premium content" }));
```

### Go (Gin)

```go
import (
    x402 "github.com/coinbase/x402/go"
    x402http "github.com/coinbase/x402/go/http"
    ginmw "github.com/coinbase/x402/go/http/gin"
    evm "github.com/coinbase/x402/go/mechanisms/evm/exact/server"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    facilitatorClient := x402http.NewHTTPFacilitatorClient(&x402http.FacilitatorConfig{
        URL: "https://api.cdp.coinbase.com/platform/v2/x402",  // Mainnet
    })

    r.Use(ginmw.X402Payment(ginmw.Config{
        Routes: x402http.RoutesConfig{
            "GET /weather": {
                Accepts: x402http.PaymentOptions{
                    {Scheme: "exact", Price: "$0.001", Network: "eip155:8453", PayTo: "0xYour"},
                },
                Description: "Weather data",
            },
        },
        Facilitator: facilitatorClient,
        Schemes: []ginmw.SchemeConfig{
            {Network: "eip155:8453", Server: evm.NewExactEvmScheme()},
        },
    }))

    r.GET("/weather", func(c *gin.Context) {
        c.JSON(200, gin.H{"weather": "sunny"})
    })
    r.Run(":4021")
}
```

### Python (FastAPI)

```python
from fastapi import FastAPI
from x402.http import FacilitatorConfig, HTTPFacilitatorClient, PaymentOption
from x402.http.middleware.fastapi import PaymentMiddlewareASGI
from x402.http.types import RouteConfig
from x402.mechanisms.evm.exact import ExactEvmServerScheme
from x402.server import x402ResourceServer

app = FastAPI()
pay_to = "0xYourAddress"

facilitator = HTTPFacilitatorClient(
    FacilitatorConfig(url="https://api.cdp.coinbase.com/platform/v2/x402")  # Mainnet
)
server = x402ResourceServer(facilitator)
server.register("eip155:8453", ExactEvmServerScheme())

routes = {
    "GET /weather": RouteConfig(
        accepts=[PaymentOption(scheme="exact", pay_to=pay_to, price="$0.001", network="eip155:8453")],
        description="Weather data",
    ),
}

app.add_middleware(PaymentMiddlewareASGI, routes=routes, server=server)

@app.get("/weather")
async def get_weather():
    return {"weather": "sunny", "temperature": 70}
```

## Multi-Network Support

Accept payments on both EVM and Solana (mainnet):

```typescript
import { registerExactEvmScheme } from "@x402/evm/exact/server";
import { registerExactSvmScheme } from "@x402/svm/exact/server";

const server = new x402ResourceServer(facilitatorClient);
registerExactEvmScheme(server);
registerExactSvmScheme(server);

app.use(
  paymentMiddleware(
    {
      "GET /data": {
        accepts: [
          { scheme: "exact", price: "$0.01", network: "eip155:8453", payTo: "0xYourEvmAddress" },  // Base Mainnet
          { scheme: "exact", price: "$0.01", network: "solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp", payTo: "YourSolanaAddress" },  // Solana Mainnet
        ],
      },
    },
    server,
  ),
);
```

## Bazaar Discovery

Make your endpoint discoverable:

```typescript
import { declareDiscoveryExtension } from "@x402/extensions/bazaar";

app.use(
  paymentMiddleware(
    {
      "GET /weather": {
        accepts: [{ scheme: "exact", price: "$0.001", network: "eip155:8453", payTo }],  // Base Mainnet
        description: "Real-time weather data",
        mimeType: "application/json",
        extensions: {
          ...declareDiscoveryExtension({
            input: { city: "San Francisco" },
            inputSchema: { properties: { city: { type: "string" } }, required: ["city"] },
            output: { example: { city: "San Francisco", weather: "foggy" } },
          }),
        },
      },
    },
    server,
  ),
);
```

## Server Payment Flow

1. Client requests protected endpoint
2. Server returns **402 Payment Required** with `PAYMENT-REQUIRED` header
3. Client creates payment and sends with `PAYMENT-SIGNATURE` header
4. Server verifies payment via Facilitator (`/verify`)
5. Server executes endpoint handler
6. Server settles payment via Facilitator (`/settle`)
7. Server returns response with `PAYMENT-RESPONSE` header

## Testing Your Server

```bash
curl -i http://localhost:4021/weather

# Expected: HTTP/1.1 402 Payment Required
# PAYMENT-REQUIRED: <base64-encoded payment requirements>
```

## Setup Checklist

1. **Get a wallet address** - Run `echo "1" | npx add-wallet evm` (non-interactive)
2. **Note the address** - Check `.env` for `WALLET_ADDRESS` to use as `payTo`
3. **Choose network** - Use `eip155:8453` (Base Mainnet) for production, or `eip155:84532` (Base Sepolia) for testing
4. **Set price** - Use dollar notation (`"$0.001"`)
5. **Configure facilitator** - Use `https://api.cdp.coinbase.com/platform/v2/x402` for mainnet (or `https://x402.org/facilitator` for testnet)
6. **Add middleware** - Protect routes with payment requirements
7. **Test** - curl should return 402 with payment instructions

## Key Packages

### TypeScript
- `@x402/express` - Express.js middleware
- `@x402/next` - Next.js middleware
- `@x402/hono` - Hono middleware
- `@x402/core` - Core server utilities
- `@x402/evm` - EVM scheme
- `@x402/svm` - Solana scheme

### Go
- `github.com/coinbase/x402/go` - Core package
- `github.com/coinbase/x402/go/http/gin` - Gin middleware

### Python
- `x402[fastapi]` - FastAPI support
- `x402[flask]` - Flask support
