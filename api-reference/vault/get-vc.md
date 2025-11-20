# Vault Get VC

Read a credential from Vault via signed XDR or direct owner payload.

## POST `/vault/get_vc`

- Signed XDR flow: `{ signedXdr: string }` → `200 { vc: unknown }`
- Direct flow: `{ owner: string; vcId: string; vaultContractId?: string }` → `200 { vc: unknown }`
- Validation errors: `owner_required`, `vcId_required`
- Errors: `500 read_error`
- Source: `api/testnet/src/controllers/vaultGetVcController.ts:6-28`