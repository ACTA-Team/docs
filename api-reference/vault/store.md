# Vault Store

Submit a signed XDR or perform server-side store to add a VC to Vault.

## POST `/vault/store`

### Signed XDR flow

- Body: `{ signedXdr: string; vcId: string; owner?: string; vaultContractId?: string }`
- Returns: `201` `{ vc_id: string; tx_id: string; verification?: { status?: string; since?: string } }`
- Errors: `400 bad_request`, `403 issuer_not_authorized`, `500 store_error`
- Source: `api/testnet/src/controllers/vaultStoreController.ts:13-43`

### Server-side flow

- Body: `{ owner: string; vcId: string; vcData: string; vaultContractId: string; issuerDid?: string; issuanceContractId?: string }`
- Returns: `201` `{ vc_id: string; tx_id: string; verification?: { status?: string; since?: string } }`
- Validations: `owner_required`, `vcId_required`, `vcData_required`, `vaultContractId_required`, `issuerDid_invalid`
- Errors: `403 issuer_not_authorized`, `400 bad_request`, `500 store_error`
- Source: `api/testnet/src/controllers/vaultStoreController.ts:46-99`