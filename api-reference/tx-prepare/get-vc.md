# Prepare Get VC

Build unsigned XDR to read a credential from Vault.

## POST `/tx/prepare/get_vc`

- Body: `{ owner: string; vcId: string; vaultContractId?: string }`
- Returns: `{ unsignedXdr: string }`
- Validation errors: `owner_required`, `vcId_required`, `bad_request` (simulation)
- Source: `api/testnet/src/controllers/txPrepareGetVcController.ts:5-23`