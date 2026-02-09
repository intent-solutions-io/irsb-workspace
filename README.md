<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/irsb-banner-dark.svg">
    <img alt="IRSB — Intent Receipts & Solver Bonds" src="assets/irsb-banner-light.svg" width="800">
  </picture>
</p>

<p align="center">
  <a href="https://github.com/intent-solutions-io/irsb/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-0ea5e9?style=flat-square" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/Solidity-0.8.25-363636?style=flat-square&logo=solidity" alt="Solidity 0.8.25">
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178c6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Ethereum-Sepolia-0ea5e9?style=flat-square&logo=ethereum&logoColor=white" alt="Ethereum Sepolia">
  <img src="https://img.shields.io/badge/ERC--8004-Agent%20%23967-38bdf8?style=flat-square" alt="ERC-8004 Agent #967">
</p>

---

> **Intent protocols like UniswapX, CoW Protocol, and 1inch Fusion route user orders through off-chain solvers — but today there are zero consequences for front-running, delayed fills, or partial execution.** IRSB adds cryptographic receipts, staked bonds, and on-chain dispute resolution so every solver action is accountable.

## How It Works

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
  'primaryColor': '#0ea5e9',
  'primaryBorderColor': '#0284c7',
  'primaryTextColor': '#ffffff',
  'lineColor': '#38bdf8',
  'secondaryColor': '#0c1929',
  'tertiaryColor': '#e5e7eb',
  'noteBkgColor': '#0c4a6e',
  'noteTextColor': '#e5e7eb',
  'actorBkg': '#0ea5e9',
  'actorBorder': '#0284c7',
  'actorTextColor': '#ffffff',
  'signalColor': '#38bdf8',
  'signalTextColor': '#e5e7eb',
  'activationBkgColor': '#0c4a6e',
  'activationBorderColor': '#0ea5e9'
}}}%%
sequenceDiagram
    participant U as User
    participant S as Solver
    participant R as IntentReceiptHub
    participant B as SolverRegistry
    participant W as Watchtower
    participant D as DisputeModule

    U->>S: Submit intent (ERC-7683)
    S->>B: Stake bond (min 0.1 ETH)
    S->>R: Execute + post receipt
    R->>R: Challenge window opens (1 hr)
    W->>R: Monitor receipts
    alt No violation
        R->>R: Receipt finalized
        S->>B: Bond returned
    else Violation detected
        W->>D: Open dispute + evidence
        D->>D: Evidence period
        D->>B: Slash bond
        Note over D,B: 80% user / 15% challenger / 5% treasury
    end
```

## Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
  'primaryColor': '#0ea5e9',
  'primaryBorderColor': '#0284c7',
  'primaryTextColor': '#ffffff',
  'lineColor': '#38bdf8',
  'secondaryColor': '#0c1929',
  'tertiaryColor': '#e5e7eb',
  'clusterBkg': '#0c192910',
  'clusterBorder': '#0ea5e9'
}}}%%
flowchart TB
    subgraph ERC8004["ERC-8004 Identity Layer"]
        REG["Identity Registry — Agent #967"]
    end

    subgraph Protocol["protocol/ — On-chain Contracts (Solidity)"]
        SR[SolverRegistry]
        IRH[IntentReceiptHub]
        DM[DisputeModule]
        EV[EscrowVault]
    end

    subgraph OffChain["Off-chain Services (TypeScript)"]
        SOL["solver/ — Reference Solver"]
        WT["watchtower/ — Monitor & Enforce"]
        AP["agent-passkey/ — PKP Signing"]
    end

    REG --> SR
    SOL -->|typed actions| AP
    WT -->|typed actions| AP
    AP -->|threshold sig| SR
    AP -->|threshold sig| IRH
    SOL -->|post receipt| IRH
    WT -->|open dispute| DM
    DM -->|slash| SR
    DM -->|release| EV
```

## Repositories

| Repo | Description | Status |
|------|-------------|--------|
| [protocol](https://github.com/intent-solutions-io/irsb-protocol) | Solidity contracts — receipts, bonds, disputes, escrow (Foundry) | Deployed (Sepolia) |
| [solver](https://github.com/intent-solutions-io/irsb-solver) | Reference solver implementation (TypeScript, Express) | Development |
| [watchtower](https://github.com/intent-solutions-io/irsb-watchtower) | Monitor receipts, detect violations, file disputes (TypeScript, Fastify) | Development |
| [agent-passkey](https://github.com/intent-solutions-io/irsb-agent-passkey) | Policy-gated signing via Lit Protocol PKP (TypeScript, Fastify) | Live (Cloud Run) |

## Live Deployments

| Service | Address / URL | Network |
|---------|---------------|---------|
| SolverRegistry | [`0xB6ab964832808E49635fF82D1996D6a888ecB745`](https://sepolia.etherscan.io/address/0xB6ab964832808E49635fF82D1996D6a888ecB745) | Sepolia |
| IntentReceiptHub | [`0xD66A1e880AA3939CA066a9EA1dD37ad3d01D977c`](https://sepolia.etherscan.io/address/0xD66A1e880AA3939CA066a9EA1dD37ad3d01D977c) | Sepolia |
| DisputeModule | [`0x144DfEcB57B08471e2A75E78fc0d2A74A89DB79D`](https://sepolia.etherscan.io/address/0x144DfEcB57B08471e2A75E78fc0d2A74A89DB79D) | Sepolia |
| Agent Passkey | [Cloud Run](https://irsb-agent-passkey-308207955734.us-central1.run.app) | GCP |
| ERC-8004 Agent | ID `967` on [IdentityRegistry](https://sepolia.etherscan.io/address/0x8004A818BFB912233c491871b3d84c89A494BD9e) | Sepolia |

## Standards

| Standard | Role in IRSB |
|----------|-------------|
| [ERC-7683](https://eips.ethereum.org/EIPS/eip-7683) | Cross-chain intent format — IRSB receipts reference ERC-7683 intent hashes |
| [ERC-8004](https://eips.ethereum.org/EIPS/eip-8004) | Trustless agent identity — IRSB publishes reputation signals to the on-chain registry |
| [x402](https://www.x402.org/) | HTTP payment protocol — IRSB solver can serve as an x402-compatible payment facilitator |

<details>
<summary><strong>Protocol Parameters</strong></summary>

| Parameter | Value |
|-----------|-------|
| Minimum Bond | 0.1 ETH |
| Challenge Window | 1 hour |
| Withdrawal Cooldown | 7 days |
| Max Jails (permanent ban) | 3 strikes |
| Counter-Bond Window | 24 hours |
| Arbitration Timeout | 7 days |

</details>

<details>
<summary><strong>Getting Started</strong></summary>

```bash
# Clone the workspace (docs + cross-cutting research)
git clone https://github.com/intent-solutions-io/irsb.git && cd irsb

# Clone individual repos into the workspace
git clone https://github.com/intent-solutions-io/irsb-protocol.git protocol
git clone https://github.com/intent-solutions-io/irsb-solver.git solver
git clone https://github.com/intent-solutions-io/irsb-watchtower.git watchtower
git clone https://github.com/intent-solutions-io/irsb-agent-passkey.git agent-passkey

# Build & test the protocol
cd protocol && forge build && forge test

# Build & test a TypeScript service
cd ../solver && pnpm install && pnpm build && pnpm test
```

Each repo has its own README and CLAUDE.md with detailed setup and contribution instructions.

</details>

## Documentation

- [AI-CONTEXT.md](./AI-CONTEXT.md) — Full ecosystem reference (contracts, concepts, glossary)
- [000-docs/](./000-docs/) — Architecture decisions and planning documents

## License

MIT
