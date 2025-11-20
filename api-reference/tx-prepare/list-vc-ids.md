# Prepare List VC IDs

Build unsigned XDR to list credential IDs in Vault.

## POST `/tx/prepare/list_vc_ids`

- Body: `{ owner: string; vaultContractId?: string }`
- Returns: `{ unsignedXdr: string }`
- Validation errors: `owner_required`, `bad_request` (simulation)
- Source: `api/testnet/src/controllers/txPrepareListVcIdsController.ts:5-20`