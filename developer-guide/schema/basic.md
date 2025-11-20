# Basic

Core primitives used by the ACTA API.

- `owner`: string — Stellar public key (G...)
- `issuer`: string — Stellar public key (G...)
- `vcId`: string — credential identifier
- `vcData`: string — serialized VC data
- `vaultContractId`: string — Soroban contract address (C...)
- `didUri`: string — DID (e.g., `did:acta:...`) (`api/testnet/src/controllers/txPrepareStoreController.ts:10-18`)
- `issuerDid`: string (optional) — DID for issuer when server-side store (`api/testnet/src/controllers/vaultStoreController.ts:51-64`)
- `signedXdr`: string — signed transaction XDR
- `fields`: object — arbitrary form fields included in store prepare (`api/testnet/src/controllers/txPrepareStoreController.ts:20-28`)

Configuration object read by server (`api/testnet/src/types/types.ts:22-30`):

- `rpcUrl`: string
- `horizonUrl`: string
- `networkPassphrase`: string
- `issuanceContractId`: string
- `secretKey`: string
- `verifySourcePublicKey?`: string
- `vaultContractId?`: string