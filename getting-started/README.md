# Getting Started

This guide explains the minimum steps to begin using `https://api.acta.build`.

## Prerequisites

- Vault and DID: Create these at `https://demo.acta.build`.
  - Vault: Holds on-chain records for your credentials.
  - DID: Decentralized identifier for the owner, aligned with W3C DID format.

## Base URL

`https://api.acta.build`

## Authentication

- Current public endpoints do not require an API key.
- Use HTTPS and handle responses and errors per the API Reference.

## Common Terms

- `owner`: The public key (G-address) of the Vault owner.
- `vaultContractId`: The smart contract address for the Vault.
- `vcId`: The identifier for the Verifiable Credential (VC), e.g., `vc:example:123`.
- `vcData`: Stringified JSON payload of the VC.
- `didUri`: Optional DID of the owner (format: `did:<method>:<id>`).
