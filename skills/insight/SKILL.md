---
name: insight
description: Check whether the price an agent is about to trade is trustworthy. Insight returns an oracle-manipulation-aware verdict (PASS / WARN / BLOCK) for a single trade, computed from multi-provider oracle consensus, with max deviation, manipulation-risk score, and an optional verifiable EIP-712 attestation. Pay-per-call $0.02 in USDC via x402 v2 — no signup, no API key, no subscription. Use before executing any swap, buy, or sell; when sizing DEX trades; when a quoted price looks off; or when the user asks "is this price safe to trade", "check for oracle manipulation", "run a pre-trade check", or asks to verify price-feed integrity. Works for trades settling on Monad, Base, Polygon, Arbitrum One, BNB Chain, and Solana.
license: MIT
metadata:
  author: imokokok
  version: "1.0"
  chain: multichain
  category: Oracles
tags:
  - insight
  - oracle
  - oracle-manipulation
  - pre-trade
  - safety
  - price-feed
  - risk
  - x402
  - defi
  - ai-agents
  - attestation
  - mcp
---

# Insight

Insight (oracleinsight.xyz) answers one question: **can this price be trusted right now?** It is an oracle safety and pre-trade risk API for DeFi and AI agents. For a single trade it returns:

- `verdict` — `PASS` / `WARN` / `BLOCK` (`BLOCK` = do not execute against the price you read)
- Consensus price and max deviation %
- Manipulation-risk score with cross-provider agreement / quorum
- Optional EIP-712 attestation any third party can re-verify offline

- **Site**: https://www.oracleinsight.xyz
- **GitHub**: https://github.com/imokokok/Insight

## What You Probably Got Wrong

- **No API key is needed** — Insight bills per call through x402 v2. There is no signup, no key, no subscription. An unauthenticated call returns `HTTP 402` with payment instructions; that is the normal flow, not an auth error.
- **A `BLOCK` verdict is still a completed check and is charged** — your agent just learned it should not trade, which is the product.
- **No native gas token is needed** — the facilitator sponsors gas on EVM chains and Solana.
- **BNB Chain USDC is different** — it settles in the 18-decimal bridged USDC via the AEON facilitator flow; the other EVM chains use standard EIP-3009 with 6-decimal USDC, and Solana uses the exact-scheme SVM flow.
- **`BLOCK` is a hard stop** — do not execute the trade against the price you read.

## When to Use

Invoke before executing any swap / buy / sell, when sizing DEX trades, when a quoted price looks off, or when the user asks:

- "is this price safe to trade"
- "check for oracle manipulation"
- "run a pre-trade check"
- "verify price-feed integrity"

Supported trade venues: Monad (`eip155:143`), Base, Polygon, Arbitrum One, BNB Chain, and Solana.

## Quick Start: Pre-Trade Safety Check

Plain HTTP:

```http
GET https://www.oracleinsight.xyz/api/v1/safety/pre-trade
    ?asset=USDC
    &chainId=8453
    &action=swap
    &tradeAmountUsd=1000
    &tokenAddress=0x...
```

| Param            | Meaning                                                                     |
| ---------------- | --------------------------------------------------------------------------- |
| `asset`          | The asset being traded (e.g. `USDC`)                                        |
| `chainId`        | EVM chain id of the trade venue (`1` Ethereum, `8453` Base, `143` Monad, …) |
| `action`         | `swap`, `buy`, `sell`, or other trade verb                                  |
| `tradeAmountUsd` | Notional size in USD                                                        |
| `tokenAddress`   | Contract address of the token being priced/traded                           |

## MCP

MCP clients call the server at `https://www.oracleinsight.xyz/api/mcp` (JSON-RPC, metered per tool class). Discover the tool schemas, then call:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/list"}
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"pre_trade_safety_check","arguments":{}}}
```

`tools/list` returns the exact argument schema for each tool, including `pre_trade_safety_check` and `verify_attestation`.

## Payment (x402 v2, exact scheme)

1. Make the call unauthenticated — it returns `HTTP 402` with a base64url `PAYMENT-REQUIRED` header listing `accepts[]`.
2. Sign one `accepts[]` entry and retry with the payment header.
3. The check runs and the response is delivered; settlement executes only after the response is deliverable.

Price: **$0.02 USDC** (`20000` atomic, 6 decimals) for the pre-trade check.

Settle in USDC on whichever network matches your wallet's inventory:

| Network      | CAIP-2               | USDC asset                                     |
| ------------ | -------------------- | ---------------------------------------------- |
| Base         | `eip155:8453`        | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`   |
| Polygon PoS  | `eip155:137`         | `0x3c499c542cef5e3811e1192ce70d8cC03d5c3359`   |
| Arbitrum One | `eip155:42161`       | `0xaf88d065e77c8cC2239327C5EDb3A432268e5831`   |
| Monad        | `eip155:143`         | `0x754704Bc059F8C67012fEd69BC8A327a5aafb603`   |
| BNB Chain    | `eip155:56`          | `0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d`   |
| Solana       | `solana:5eykt4…Kvdp` | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |

Full discovery manifest: `GET https://www.oracleinsight.xyz/.well-known/x402`

## Billing Semantics

- A `BLOCK` verdict is a completed check and is charged.
- Settlement executes only after the check completes and the response is deliverable.
- No chargebacks; failed payments produce no response body.

| HTTP response | Meaning            | Fix                                                        |
| ------------- | ------------------ | ---------------------------------------------------------- |
| `402`         | Payment required   | Sign one `accepts[]` entry and retry with the payment header |
| No body       | Payment failed     | Nothing was delivered; re-attempt the payment               |

## Attestation

Checks can return an optional EIP-712 attestation — a signed statement about the verdict that any third party can re-verify offline. Use the `verify_attestation` tool over MCP to verify an attestation you received.

## Security Considerations

- Treat `BLOCK` as a hard stop: do not execute the trade against the price you read.
- Verify EIP-712 attestations offline before relying on them in automated flows.
- Insight is a safety oracle: it does not custody funds, execute trades, or provide investment advice.

## References

- [Site](https://www.oracleinsight.xyz)
- [GitHub](https://github.com/imokokok/Insight)
- [OpenAPI 3.1 contract](https://www.oracleinsight.xyz/openapi.json)
- [x402 discovery manifest](https://www.oracleinsight.xyz/.well-known/x402)
- [Full manual](https://www.oracleinsight.xyz/skill.md)
