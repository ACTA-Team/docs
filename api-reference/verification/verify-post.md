# Verify Credential (POST)

Verify VC status via Vault or Issuance depending on payload.

## POST `/verify`

- Owner payload: `{ owner: string; vcId: string; vaultContractId?: string }` → `{ vc_id: string; status: string; since?: string }`
- Without owner: `{ vcId: string }` → `{ vc_id: string; status: string; since?: string }`
- Validation errors: `vcId_required`
- Errors: `500 verify_error`
- Source: `api/testnet/src/controllers/verifyController.ts:14-36`