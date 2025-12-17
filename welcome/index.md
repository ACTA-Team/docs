# Introduction

ACTA is a verifiable credentials infrastructure on Stellar blockchain that provides on-chain credential status and encrypted off-chain storage.

## Overview

ACTA enables the issuance, storage, and verification of W3C Verifiable Credentials 2.0 on the Stellar network using Soroban smart contracts. Credentials are anchored on-chain for public verification while maintaining encrypted off-chain data storage.

## Key Components

### Issuance Contract

Smart contract that issues credentials, tracks their status, and handles revocation. Status changes are publicly verifiable on-chain.

### Vault Contract

Multi-tenant repository for storing encrypted credentials. Each user has an isolated vault with independent issuer authorization.

### API

RESTful API providing endpoints for credential operations, transaction preparation, and vault management. Supports both mainnet and testnet.

### SDK

TypeScript/React SDK for easy integration with web applications and dApps.

## Architecture

- **Identity**: DID:pkh format using Stellar wallet addresses (`did:pkh:stellar:{network}:{address}`)
- **On-chain**: Credential hashes and status anchored on Soroban for public verification
- **Off-chain**: Encrypted credential data stored in user vaults
- **Verification**: Instant status checks via on-chain hash verification

## Standards Compliance

- W3C Verifiable Credentials 2.0
- W3C VC Data Model
- JSON-LD compatible
- DID:pkh specification

## Getting Started

- [API Quickstart](../api-reference/developer-quickstart.md) - Start using the API
- [React SDK](../react-sdk/index.md) - Integrate with React applications
- [Developer Guide](../developer-guide/index.md) - Detailed integration guides
