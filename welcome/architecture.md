# Architecture

Technical overview of ACTA's system architecture and components.

## System Components

### Issuance Contract (Soroban)

Handles credential lifecycle on-chain:

- **Issue**: Creates new credentials and anchors hash on-chain
- **Verify**: Public verification of credential status
- **Revoke**: Revokes credentials with optional revocation date

Contract functions are exposed via API endpoints. See [API Reference](../api-reference/index.md) for details.

### Vault Contract (Soroban)

Multi-tenant credential storage repository:

- **Initialize**: Creates a new vault for a user
- **Store**: Stores encrypted credentials in the user's vault
- **List/Get**: Retrieves credential IDs and data
- **Verify**: Verifies credentials via issuance contract delegation
- **Push**: Transfers credentials between vaults
- **Authorization**: Manages issuer authorization lists

Each user has an isolated vault with independent admin controls and issuer authorization.

### API Layer

RESTful API providing:

- **Credential Operations**: Issue, verify, revoke
- **Vault Operations**: Store, retrieve, manage vaults
- **Transaction Preparation**: Generate unsigned XDR transactions for client-side signing
- **Read Operations**: Query credentials and vault state (no signature required)

All endpoints support both mainnet and testnet automatically via `NETWORK_TYPE` configuration.

### Storage

- **On-chain**: Credential hashes and status metadata (Soroban smart contracts)
- **Off-chain**: Encrypted credential payloads (user-controlled vaults)

## Identity Model

Uses DID:pkh format:

```
did:pkh:stellar:{network}:{wallet_address}
```

- **Network**: `mainnet` or `testnet`
- **Wallet Address**: Stellar public key (G...)

No additional identity infrastructure required - Stellar wallet keys serve as identity.

## Credential Flow

### Issuance Flow

1. Issuer calls API with credential data
2. API canonicalizes and creates W3C VC
3. VC hash anchored on Issuance contract
4. Encrypted VC data stored in holder's Vault
5. Credential ID returned to issuer

### Verification Flow

1. Verifier queries credential status via API
2. API checks Issuance contract on-chain status
3. Status returned (valid, revoked, invalid)
4. Optional: Retrieve full credential data from Vault

### Storage Flow

1. Credentials stored in holder's Vault contract
2. Each vault is isolated per owner address
3. Vault admin controls issuer authorization
4. Only authorized issuers can store in vault
5. Credentials encrypted and accessible only to owner

## Network Support

ACTA automatically handles network configuration:

- **Testnet**: `https://acta.build/api/testnet` or `NETWORK_TYPE=testnet`
- **Mainnet**: `https://acta.build/api/mainnet` or `NETWORK_TYPE=mainnet`

Contract IDs, RPC URLs, and network passphrases are configured automatically based on network type.

