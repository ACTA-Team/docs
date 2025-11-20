# Verify Credential (GET)

Verify VC status via the Issuance contract.

## GET `/verify/:vc_id`

- Path: `vc_id`
- Returns: `{ vc_id: string; status: string; since?: string }`
- Errors: `500 verify_error`
- Source: `api/testnet/src/controllers/verifyController.ts:5-13`