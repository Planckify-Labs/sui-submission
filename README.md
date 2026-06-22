# TakumiPay: The Intent Engine for Sui

**Tell it what you want. It builds the transaction, warns you before you sign, and executes atomically on Sui.** Live on Sui **mainnet** today.

[![Track](https://img.shields.io/badge/Track-Agentic%20Web%20·%20Intent%20Engine-6f4cff)](#) [![Network](https://img.shields.io/badge/Sui-Mainnet%20%2B%20Testnet-4da2ff)](#on-chain-proof) [![Demo](https://img.shields.io/badge/▶-3--min%20demo-ff0000)](https://youtu.be/oGqQZ5wMDuM)

- **▶ Demo (3 min):** https://youtu.be/oGqQZ5wMDuM
- **Track:** The Agentic Web → Sub-track 3 (Intent Engine) · Core Track
- **Network:** Sui **Mainnet** (also runs on Testnet) · **Package ID** [`0x68e6…0f03`](https://suivision.xyz/package/0x68e6de85ba7178056ca70c4900e9cb3d87838248d83334a1b8e16ffd8dcb0f03)
- **Website:** https://takumipay.xyz

---

## At a glance

- **What** — Say a financial goal in plain language; the Takumi agent compiles it into **one Sui PTB**; a **guardian** flags the risks before you sign; you confirm with a native biometric sheet; it executes atomically and returns an on-chain digest.
- **Why Sui** — PTBs are the agent's compile target. Before signing, the guardian runs `dryRunTransactionBlock` and inspects the **exact object/pool state and balance changes *this* transaction will produce** — real risk from the real transaction, not a heuristic. Sui isn't a bolted-on payment rail; it's what makes the AI safer.
- **Why it's real** — Ships inside a **live, mainnet mobile wallet with real users**. The agent and guardian speak **plain language in any language** (it's an LLM agent), making DeFi usable for people who don't speak DeFi — financial inclusion, not a crypto toy.

## Demo

**▶ Watch the 3-minute demo:** https://youtu.be/oGqQZ5wMDuM

What you'll see: a plain-language goal → a **decoded PTB preview** + guardian risk rows → confirm a *safe* intent (→ on-chain digest) → then a deliberately *risky* intent that the guardian **blocks before it can be signed**.

<!-- TODO: add real screenshots (pull frames from the demo). Suggested:
![Intent preview + guardian rows](docs/screenshots/intent-preview.png)
![Guardian blocking a risky intent](docs/screenshots/guardian-block.png)
-->
> <img width="727" height="1600" alt="image" src="https://github.com/user-attachments/assets/672202c1-40c8-4bde-97e0-625a0fe5cd82" />


## Why Sui specifically

A multi-step goal becomes **one atomic, auditable, previewable transaction**. Before signing, the
guardian runs `dryRunTransactionBlock` and inspects the exact object/pool state and the precise
balance/object changes this transaction will produce — so the risk shown is computed from the **real
effects of the real transaction**, not a generic heuristic. Sui's object model + PTB atomicity +
dry-run effects are what make the guardian possible and trustworthy. Sui isn't a payment rail bolted
on at the end here; it's what makes the AI **safer**.

TakumiPay is deliberately chain-agnostic (EVM, Solana, Sui) — which is exactly what makes this a
*deliberate* choice. We didn't build the Intent Engine on the chain that happens to hold the token;
we built it on **Sui** because only its PTB atomicity + `dryRunTransactionBlock` effects make the
guardian possible. Knowing EVM and Solana intimately is what tells us Sui's object model is the one
that makes the AI safer.

### Real protocols, wired today

The engine executes against **live Sui DeFi, not mocks** — both shown in the demo:

- **Swaps → DeepBook v3** — Mysten's on-chain central limit order book. The agent pulls a real quote (which also feeds the slippage guardian) and routes the swap through DeepBook v3.
- **Yield (supply / withdraw) → Scallop** — Sui's lending money market, via its official SDK.

## The guardian — the make-or-break

The sub-track explicitly rejects "a swap chatbot with no guardian." Ours inspects the compiled PTB and
its dry-run effects and flags **three risk classes** in plain language — the three the track names:

| Risk class | Fires when |
|---|---|
| **High slippage / price impact** | a large order against a thin pool would lose too much to price impact |
| **Stale pool / oracle** | the pool's price/state hasn't updated within a freshness threshold |
| **Over-concentration** | the action pushes too much of your funds into a single venue/asset |

On top of the three, an **execution-integrity safeguard** compares the dry-run's real balance/object
changes against what was previewed and refuses on any mismatch — so what you approve is exactly what
signs.

A risky intent like *"swap 90% of my SUI now"* is **blocked before it ever reaches the signing
sheet** — and the `write` tool **re-runs the guardian at execute time**, so a blocked intent can never
be signed even if the model misbehaves. The decline path is first-class and demoed.

## How it works

```
plain-language goal (any language)
  → structured intent (zod)                     defi specialist emits a validated Intent
  → Sui PTB (@mysten/sui)                        compiler dispatches to Scallop (supply) or DeepBook/Cetus/7K (swap)
  → decoded preview + guardian risk flags        dryRunTransactionBlock inspects the exact effects
  → explicit confirmation                        native biometric approval sheet (the eyes-open gate)
  → execution                                    on-device signer submits the sui-ptb
  → on-chain digest + intent_receipt record      every executed intent is logged on-chain
```

Two agent tools split the flow so the guardian can read live state **before** anything is signable:
`defi_intent_preview` (read — compile + guard) → `defi_intent_execute` (write — sign + submit + record).

## Where the Sui code lives

The heart of "meaningful Sui integration" — direct links:

| Concern | Repo · path |
|---|---|
| Structured intent schema (zod) | mobile-app · [`services/chains/sui/intent/intentSchema.ts`](https://github.com/Planckify-Labs/sui-submission-mobile-app/blob/main/services/chains/sui/intent/intentSchema.ts) |
| NL → PTB compiler | mobile-app · [`services/chains/sui/intent/compileIntentToPtb.ts`](https://github.com/Planckify-Labs/sui-submission-mobile-app/blob/main/services/chains/sui/intent/compileIntentToPtb.ts) |
| Human-readable PTB decode | mobile-app · [`services/chains/sui/intent/decodeSuiPtb.ts`](https://github.com/Planckify-Labs/sui-submission-mobile-app/blob/main/services/chains/sui/intent/decodeSuiPtb.ts) |
| Guardian + risk checks | mobile-app · [`services/chains/sui/intent/guardian/`](https://github.com/Planckify-Labs/sui-submission-mobile-app/tree/main/services/chains/sui/intent/guardian) |
| Swap routing (DeepBook · Cetus · 7K) | mobile-app · [`services/swap/sui/`](https://github.com/Planckify-Labs/sui-submission-mobile-app/tree/main/services/swap/sui) |
| On-chain receipt (append + package-id resolve) | mobile-app · [`services/swap/sui/appendIntentReceipt.ts`](https://github.com/Planckify-Labs/sui-submission-mobile-app/blob/main/services/swap/sui/appendIntentReceipt.ts) |
| Agent executors (preview + execute) | mobile-app · [`services/agent-executors/defi/intentExecutors.ts`](https://github.com/Planckify-Labs/sui-submission-mobile-app/blob/main/services/agent-executors/defi/intentExecutors.ts) |
| Agent tool definitions | agent-api · [`src/agents/defi/tools/intent.ts`](https://github.com/Planckify-Labs/sui-submission-agent-api/blob/main/src/agents/defi/tools/intent.ts) |
| Move audit-log module | contracts · [`intent_receipt.move`](https://github.com/Planckify-Labs/sui-submission-contracts) |

## On-chain proof

TakumiPay is **deployed and running on Sui mainnet**.

| What | Network | Link |
|---|---|---|
| Example intent executed by the engine | Mainnet | <!-- TODO --> _add a SuiVision digest link from the demo run_ |
| `intent_receipt` package | **Mainnet** | [`0x68e6…0f03`](https://suivision.xyz/package/0x68e6de85ba7178056ca70c4900e9cb3d87838248d83334a1b8e16ffd8dcb0f03) · [publish tx](https://suivision.xyz/txblock/4gCAq4ZdQiJuSyjhyXpHH7W4DM2MFZ64faxUszQv9ota) |
| `intent_receipt` package | Testnet | [`0x0bea…a301`](https://suivision.xyz/package/0x0bea3f1e47e213a95dc3d47148ace7047310e2d14dbc10dcb9eda6226a4ba301?network=testnet) |

## Repositories

Four repos under the [Planckify-Labs](https://github.com/Planckify-Labs) org:

| Repository | Role |
|---|---|
| [**sui-submission-mobile-app**](https://github.com/Planckify-Labs/sui-submission-mobile-app) | React Native + Expo wallet — Takumi agent, NL→PTB compiler, guardian, swap routing, on-device signer. |
| [**sui-submission-agent-api**](https://github.com/Planckify-Labs/sui-submission-agent-api) | NestJS + Vercel AI SDK agent — source of truth for tool definitions (`defi_intent_preview`, `defi_intent_execute`). |
| [**sui-submission-api**](https://github.com/Planckify-Labs/sui-submission-api) | NestJS REST API — DeFi reads, strategy/policy signals, per-network `intent_receipt` Package-ID resolution. |
| [**sui-submission-contracts**](https://github.com/Planckify-Labs/sui-submission-contracts) | Sui Move package — the `intent_receipt` audit-log module. Deployed to mainnet + testnet. |

## Run it

```bash
# Mobile app
cd sui-submission-mobile-app && pnpm install && pnpm start   # Expo dev server

# Agent API
cd sui-submission-agent-api && pnpm install && pnpm start:dev

# REST API
cd sui-submission-api && pnpm install && pnpm start:dev

# Move package
cd sui-submission-contracts && sui move build && sui move test
```

Point the mobile app at your API/agent URLs (`EXPO_PUBLIC_API_URL`, `EXPO_PUBLIC_AI_API_URL`) — see
each repo's `.env.example`.

## Real-world impact

Mass adoption happens on a phone, not a desktop browser extension: **70%+ of crypto users access
wallets via mobile in 2025**, and Asia-Pacific is the largest region by active wallets. TakumiPay
ships the intent engine **inside one native app** — wallet, AI agent, and signer together — so there's
no browser-extension juggling, and the guardian's risks appear on a native biometric sheet rather than
a raw-calldata popup. Because the agent speaks any language, the same product serves every mobile-first
emerging market (our launch market, Indonesia, is the #1 crypto-adoption market in SE Asia — 7th
globally, Chainalysis 2025).

## Roadmap — Phase 2: Autonomous Risk Guardian

The same guardian + PTB engine, with the confirm step replaced by **autonomous execution inside an
on-chain leash** — a Move **Agent Authority** object (budget · position-scope · expiry · revocable).
Hero use case: liquidation protection on Scallop — the agent de-risks your position 24/7 but **can
never withdraw funds externally**. Human-confirmed today; chain-enforced autonomy tomorrow.

## Team & submission

- **Team:** <!-- TODO: list ≥2 members + roles -->
- **KYC:** at least one team member can complete KYC to receive the prize.
- **Built during** Sui Overflow 2026 — substantial new functionality (the Sui intent layer, guardian,
  and `intent_receipt` Move package) built during the hackathon period.

---

<sub>Note: code-map deep links assume default branch `main` and dev-tree paths — verify they resolve after each repo is pushed.</sub>
