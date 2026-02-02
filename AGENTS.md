# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

A collection of skills for AI agents to interact with the x402 payment protocol. x402 enables programmatic USDC payments over HTTP, allowing agents to autonomously buy and sell services.

## Skills

### buy
Enables agents to pay for x402-protected endpoints. Handles 402 responses, signs payments, and retries requests.

### sell
Enables agents to payment-gate their own endpoints. Adds middleware to accept USDC payments via CDP Facilitator.

## Skill Structure

```
skills/
  buy/                    # Buyer/client skill
    SKILL.md              # Main instructions
    references/           # Protocol documentation
      wallet-setup.md     # Wallet creation guide (incl. non-interactive CLI)
      v1-protocol.md
      v2-protocol.md
  sell/                   # Seller/server skill
    SKILL.md              # Main instructions
    references/           # Protocol documentation
      wallet-setup.md     # Wallet creation guide (incl. non-interactive CLI)
```

## SKILL.md Format

```markdown
---
name: {skill-name}
description: {When to use this skill. Include trigger phrases.}
---

# {Skill Title}

{Instructions for the agent}
```

## Key Concepts

### x402 Protocol
- HTTP 402 status code indicates payment required
- Payment requirements sent via `PAYMENT-REQUIRED` header
- Payments signed and sent via `PAYMENT-SIGNATURE` header
- Settlement confirmed via `PAYMENT-RESPONSE` header

### Networks (CAIP-2 Format)
- Base Mainnet: `eip155:8453`
- Base Sepolia: `eip155:84532`
- Solana Mainnet: `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`
- Solana Devnet: `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1`

### Facilitators
- Testnet: `https://x402.org/facilitator`
- Mainnet: `https://api.cdp.coinbase.com/platform/v2/x402`

## End-User Installation

```bash
npx skills add Must-be-Ash/x402-skills --skill buy
npx skills add Must-be-Ash/x402-skills --skill sell
```
