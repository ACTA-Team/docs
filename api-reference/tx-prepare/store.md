# Prepare Store Transaction

Build unsigned XDR to store a VC in Vault.

## POST `/tx/prepare/store`

- Body: `{ owner: string; vcId: string; didUri: string; fields: Record<string, unknown>; vaultContractId?: string; issuer?: string }`
- Returns: `{ unsignedXdr: string }`
- Validation errors: `owner_required`, `vcId_required`, `didUri_required`, `bad_request` (simulation)
- Source: `api/testnet/src/controllers/txPrepareStoreController.ts:5-45`