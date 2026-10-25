# ChainTrace

# ChainTrace

ChainTrace is a personal multi-chain blockchain fund-flow analysis and investigation project designed to normalize, analyze, and trace on-chain asset movements across different blockchain architectures.

The project focuses on practical investigation workflows, including fund-flow analysis, downstream tracing, multi-hop path discovery, evidence preservation, and cross-chain transaction resolution.

## Current Support

| Network | Assets | Flow Analysis | One-Hop | Multi-Hop |
| --- | --- | --- | --- | --- |
| Ethereum | ERC-20 | ✅ | ✅ | ✅ |
| Arbitrum | ERC-20 | ✅ | ✅ | ✅ |
| TRON | TRC-20 / TRX | ✅ | ✅ | ✅ |
| Solana | SPL Token / SOL | ✅ | ✅ | ✅ |

## Core Capabilities

### Fund Flow Analysis

ChainTrace normalizes chain-specific transaction data into a common analytical model and supports:

- Inflow
- Outflow
- Net Flow
- Counterparty Analysis
- Native Asset and Token separation
- Exact amount handling without floating-point calculations

### One-Hop Tracing

One-Hop tracing identifies bounded downstream candidates after a known source transfer while preserving:

- Asset identity
- Time-window constraints
- Amount rules
- Chain-specific acquisition coverage
- Evidence and completeness metadata

A downstream candidate does not automatically mean that the same funds have been conclusively traced.

### Multi-Hop Tracing

The existing bounded Multi-Hop engine supports:

- Breadth-first traversal
- Path-local cycle prevention
- Account / contract / program classification
- Query and depth bounds
- Partial-result preservation
- Completeness and stop-reason reporting

Chain-specific execution models are handled through adapters rather than separate tracing engines.

## Multi-Chain Architecture

ChainTrace currently includes adapters and normalization logic for several blockchain architectures:

### EVM

Ethereum and Arbitrum use EVM-compatible acquisition and address / contract classification.

### TRON

TRON supports:

- TRC-20 transfers
- Native TRX transfers
- Base58Check address normalization
- TRON account classification
- One-Hop and Multi-Hop tracing

### Solana

Solana support includes:

- Transaction signature acquisition
- Versioned transaction parsing
- SPL Token and Token-2022 transfers
- Native SOL transfers
- Token Account and Owner separation
- Inner instruction / CPI parsing
- Owner-aware One-Hop and Multi-Hop acquisition

The system explicitly distinguishes Token Accounts, owners, programs, and other Solana account roles to avoid treating them as equivalent entities.

## Canonical Data Model

ChainTrace uses a cross-chain canonical representation for:

- `network_identifier`
- `transaction_id`
- `event_id`
- chain position (`block_number` / `slot`)
- asset identity
- native assets
- token assets
- address roles
- timestamps
- exact amounts
- query coverage and completeness

Chain-native identifiers and semantics are preserved instead of forcing every chain into an EVM-style model.

## Cross-Chain Resolution

Cross-chain resolution is currently under development.

The core resolution layer already distinguishes:

- `RESOLVED`
- `PENDING`
- `AMBIGUOUS`
- `UNRESOLVED`
- `UNSUPPORTED`

Evidence is separated into:

- `FACT`
- `INFERENCE`
- `HEURISTIC`

A cross-chain path cannot be marked `RESOLVED` using amount similarity, timestamp proximity, or other heuristic evidence alone. Deterministic protocol-native evidence is required.

Protocol-specific resolvers are being developed separately.

## Investigation Principles

ChainTrace follows several conservative evidence rules:

- A source address is not automatically a victim.
- A downstream candidate is not automatically the same funds.
- Pending evidence is not treated as resolved.
- Router, swap, bridge, refund, and self-funded transformations must be checked before loss attribution.
- Cross-chain links must not create duplicate transfer records.
- Historical coverage limitations must remain visible in analysis output.

## Development Status

The project has completed the core multi-chain architecture for:

```
Ethereum
Arbitrum
TRON
Solana
```

Current development is focused on protocol-specific cross-chain resolution.

The automated test suite currently contains more than 1,200 passing tests covering adapters, normalization, Flow, One-Hop, Multi-Hop, canonical models, evidence handling, and regression behavior.

## Project Purpose

ChainTrace is being developed as both:

1. a practical tool for blockchain fund-flow investigations, and
2. a technical portfolio project exploring blockchain data, AML-oriented investigations, and cross-chain evidence analysis.

## Disclaimer

ChainTrace is an independent personal research and development project.

It is not affiliated with Ethereum, Arbitrum, TRON, Solana, Across, or any other blockchain protocol or organization mentioned in this repository.

Analysis results should be treated as investigative evidence and technical output, not as definitive attribution of ownership, intent, criminal activity, or legal responsibility.