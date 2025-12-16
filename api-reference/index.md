# API Reference

Complete reference documentation for the ACTA API REST endpoints.

## Base URLs

- Mainnet: `https://acta.build/api/mainnet`
- Testnet: `https://acta.build/api/testnet`
- Local dev: `http://localhost:8000`

## Network Support

All endpoints automatically work with both **mainnet** and **testnet** based on the API instance configuration. The network type is determined by the `NETWORK_TYPE` environment variable.

## API Sections

### Core Endpoints

- **Health** - API health check
- **Credentials** - Issue verifiable credentials
- **Verification** - Verify credential status
- **Issuance** - Issuance contract operations (revoke, version)

### Vault Operations

The Vault contract provides secure storage for verifiable credentials:

- **Initialize** - Create a new vault for an owner
- **Store** - Store a credential in a vault
- **Get VC** - Retrieve a credential from a vault (read-only)
- **List VC IDs** - List all credential IDs in a vault (read-only)
- **Verify VC** - Verify a credential status via vault (read-only)
- **Push** - Move a credential between vaults
- **Authorize Issuer** - Authorize an issuer to store credentials (single)
- **Authorize Issuers** - Authorize multiple issuers at once
- **Revoke Issuer** - Revoke issuer authorization
- **Revoke Vault** - Disable an entire vault
- **Version** - Get vault contract version (read-only)

### Transaction Preparation

Endpoints that prepare unsigned XDR transactions for client-side signing:

- **Prepare Issue** - Prepare transaction to issue a credential
- **Prepare Store** - Prepare transaction to store a credential
- **Prepare Initialize** - Prepare transaction to initialize a vault
- **Prepare Authorize Issuer** - Prepare transaction to authorize issuer(s)
- **Prepare Push** - Prepare transaction to move credential
- **Prepare Revoke Vault** - Prepare transaction to revoke vault

### Read-Only Operations

These operations don't require signatures and use transaction simulation:

- `/vault/get_vc` - Get credential from vault
- `/vault/list_vc_ids` - List credential IDs
- `/vault/verify_vc` - Verify credential via vault
- `/verify` - Verify credential via issuance contract
- `/vault/version` - Get vault contract version
- `/issuance/version` - Get issuance contract version

## Common Patterns

### Signed XDR Flow

Many endpoints support submitting pre-signed transactions:

1. Client prepares transaction using `/tx/prepare/*` endpoints
2. Client signs the XDR locally
3. Client submits signed XDR to the main endpoint (e.g., `/vault/store`)

### Direct Flow

Some endpoints support direct execution when `STELLAR_SECRET_KEY` is configured:

1. Client sends request with operation parameters
2. Server signs and submits transaction automatically

### Read-Only Flow

Read-only operations don't require signatures:

1. Client sends request with query parameters
2. Server uses transaction simulation to read data
3. Server returns data without submitting transaction