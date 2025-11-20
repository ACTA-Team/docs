# Vault Revoke Issuer

Revoke an issuer authorization in the Vault contract via signed XDR or direct inputs.

## POST `/vault/revoke_issuer`

- Signed XDR flow: `{ signedXdr: string }` → `201 { tx_id: string }`
- Direct flow: `{ owner: string; issuer: string; vaultContractId: string }` → `201 { tx_id: string }`
- Validation errors: `owner_required`, `issuer_required`, `vaultContractId_required`
- Errors: `400 bad_request`, `500 revoke_error`
- Source: `api/testnet/src/controllers/vaultRevokeIssuerController.ts:4-41`