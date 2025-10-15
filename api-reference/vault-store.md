# POST /vault/store

Stores a credential directly in the Vault. The API signs the transaction (server-signed); the issuer defaults to ACTA’s public key (`API_PUBLIC_KEY`).

- Method: `POST`
- URL: `https://api.acta.build/vault/store`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...OWNER_PUBLIC_KEY...",
  "vaultContractId": "C...VAULT_CONTRACT_ADDRESS...",
  "vcId": "vc:example:123",
  "vcData": "{\"type\":\"Attestation\",\"subject\":\"...\"}",
"issuerDid": "did:pkh:<issuer>",
  "issuanceContractId": "<optional-issuance-contract-address>"
}
```

Success (201):
```json
{ "vc_id": "vc:example:123", "tx_id": "<transaction-hash>" }
```

Errors:
- 400 `bad_request` — validation or simulation error
- 403 `issuer_not_authorized` — issuer not authorized in owner’s Vault
- 500 `store_error` — internal processing error