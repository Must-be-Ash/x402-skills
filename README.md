# x402 Skills

Skills for AI agents to buy and sell using the x402 payment protocol. Enable autonomous micropayments over HTTP with USDC on Base and Solana.

Skills follow the [Agent Skills](https://skills.sh/) format.

## Available Skills

### buy

Enable AI agents to pay for x402-protected API endpoints. Handles HTTP 402 responses, creates payment signatures, and retries requests automatically.

**Use when:**
- Calling an API that returns HTTP 402 Payment Required
- Building agents that need to pay for resources
- Accessing x402-enabled services
- Discovering available paid services via Bazaar

**Features:**
- Automatic 402 handling with payment retry
- EVM (Base, Ethereum, Avalanche) and Solana support
- Service discovery via Bazaar
- Payment policies and lifecycle hooks

### sell

Enable AI agents to accept USDC payments for API endpoints. Payment-gate your services with middleware for Express, Next.js, Hono, Go, and Python.

**Use when:**
- Creating paid API endpoints
- Monetizing agent services with micropayments
- Building AI agents that sell data or services
- Setting up payment middleware

**Features:**
- Framework middleware (Express, Next.js, Hono, Go, Python)
- CDP Facilitator integration for settlement
- Dynamic pricing support
- Bazaar discovery for discoverability

## Installation

```bash
npx skills add Must-be-Ash/x402-skills --skill buy
npx skills add Must-be-Ash/x402-skills --skill sell
```

Or install both:

```bash
npx skills add Must-be-Ash/x402-skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**
```
Pay for this API endpoint
```
```
Add payment protection to my /api/premium route
```
```
Find x402 services that provide weather data
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent
- `references/` - Supporting documentation (optional)

## Links

- [x402 Protocol](https://x402.org)
- [x402 SDK](https://github.com/coinbase/x402)
- [Bazaar Discovery](https://x402.org/ecosystem)

## License

MIT
# x402-skills
