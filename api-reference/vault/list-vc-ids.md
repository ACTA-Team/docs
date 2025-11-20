# Vault List VC IDs

List credential IDs in Vault via signed XDR or direct owner payload.

## POST `/vault/list_vc_ids`

- Signed XDR flow: `{ signedXdr: string }` → `200 { vc_ids: string[] }`
- Direct flow: `{ owner: string; vaultContractId?: string }` → `200 { vc_ids: string[] }`
- Validation errors: `owner_required`
- Errors: `500 unexpected_return_shape`, `500 read_error`
- Source: `api/testnet/src/controllers/vaultListVcIdsController.ts:6-27`