# CLAUDE.md

> **AI Context**: For ecosystem-wide reference (contracts, deployments, concepts, glossary), see [AI-CONTEXT.md](./AI-CONTEXT.md)

This file provides guidance to Claude Code when working with the IRSB ecosystem.

## Workspace Overview

**IRSB (Intent Receipts & Solver Bonds)** is Ethereum's accountability layer for intent-based transactions. This workspace contains all IRSB projects in a unified structure.

## Project Structure

```
irsb/
├── protocol/        # On-chain Solidity contracts
├── solver/          # Reference off-chain solver
├── watchtower/      # Monitoring and dispute service
└── agent-passkey/   # Policy-gated signing gateway (NEW)
```

## Project Quick Reference

| Project | Tech Stack | Primary Use | Status |
|---------|------------|-------------|--------|
| `protocol/` | Solidity, Foundry | Smart contracts for receipts, bonds, disputes | ✅ Deployed (Sepolia) |
| `solver/` | TypeScript, Node.js | Execute intents, submit receipts | 🚧 Development |
| `watchtower/` | TypeScript, Node.js | Monitor receipts, file disputes | ✅ v0.3.0 |
| `agent-passkey/` | TypeScript, Fastify | Policy-gated signing (Lit Protocol) | ✅ Live |

## Live Deployments

| Service | URL | Network |
|---------|-----|---------|
| Agent Passkey | https://irsb-agent-passkey-308207955734.us-central1.run.app | GCP Cloud Run |
| Protocol Contracts | See `protocol/deployments/` | Sepolia |

**GCP Project:** `irsb-protocol` (project number: 308207955734)

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  ERC-8004 (Registry Layer) - Agent identity & reputation       │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────────────┐
│  IRSB Protocol (protocol/) - On-chain accountability           │
│  - Intent receipts, solver bonds, dispute resolution           │
└──────────┬────────────────────────────┬─────────────────────────┘
           │                            │
┌──────────┴──────────┐    ┌────────────┴────────────┐
│  Solver (solver/)   │    │  Watchtower (watchtower/)│
│  - Execute intents  │    │  - Monitor receipts      │
│  - Submit receipts  │    │  - File disputes         │
└──────────┬──────────┘    └────────────┬────────────┘
           │                            │
           └──────────┬─────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────────────┐
│  Agent Passkey (agent-passkey/) - Identity plane               │
│  - Lit Protocol PKP (2/3 threshold signatures)                 │
│  - Policy engine, session capabilities, audit artifacts        │
└─────────────────────────────────────────────────────────────────┘
```

## Common Commands

### Protocol (Foundry/Solidity)

```bash
cd protocol/
forge build           # Compile contracts
forge test            # Run tests
forge script ...      # Deploy scripts
```

### Solver

```bash
cd solver/
pnpm install
pnpm test             # Run tests
pnpm dev              # Development mode
```

### Watchtower

```bash
cd watchtower/
pnpm install
pnpm test             # Run tests
pnpm dev              # Development mode
```

### Agent Passkey

```bash
cd agent-passkey/
pnpm install
pnpm test             # Run tests
pnpm dev              # Development server
```

## Cross-Project Dependencies

```
protocol → (ABI/types) → solver, watchtower
agent-passkey → (signing client) → solver, watchtower
```

When making changes:
1. Update `protocol/` first if contract interfaces change
2. Regenerate types for TypeScript projects
3. Update `agent-passkey/` if signing interface changes
4. Update `solver/` and `watchtower/` last

## Task Tracking (Beads)

Each project has its own beads configuration. Use the project-level `bd` commands:

```bash
# In any project directory
bd ready              # Available tasks
bd list --status in_progress
bd sync               # Sync with git
```

## Key Concepts

### Intent Receipts
Cryptographic proof that a solver executed an intent correctly. Submitted on-chain with evidence hash.

### Solver Bonds
Staked collateral that can be slashed if a solver misbehaves. Creates economic accountability.

### Disputes
Watchtowers can challenge receipts by opening disputes. Arbitration determines if the solver's bond is slashed.

### Agent Passkey
Policy-gated signing service using **Lit Protocol PKP** (Programmable Key Pairs). Agents submit high-level actions (SUBMIT_RECEIPT, OPEN_DISPUTE, SUBMIT_EVIDENCE), not raw transaction data. The service:
- Validates against policy (allowlists, limits, role auth)
- Builds complete transactions (owns nonce management)
- Signs with Lit Protocol threshold signatures (2/3 TEE nodes)
- Produces deterministic audit artifacts

**Why Lit Protocol?** Non-extractable keys distributed across decentralized TEE nodes. No single point of key compromise. Aligns with blockchain's trust model.

## Environment Setup

### GCP Authentication

All projects use GCP services:

```bash
gcloud auth application-default login
gcloud config set project your-project-id
```

### Local Development

Each project has its own `.env.example`. Copy and configure:

```bash
cp .env.example .env
```

## Documentation

Each project has a `000-docs/` directory with:
- ADRs (Architecture Decision Records)
- Specifications
- Guides

## GitHub Repositories

All repos are under `intent-solutions-io`:
- [irsb-protocol](https://github.com/intent-solutions-io/irsb-protocol)
- [irsb-solver](https://github.com/intent-solutions-io/irsb-solver)
- [irsb-watchtower](https://github.com/intent-solutions-io/irsb-watchtower)
- [irsb-agent-passkey](https://github.com/intent-solutions-io/irsb-agent-passkey)
