# POST /credentials

Issues a credential via the issuance contract and stores it in the Vault.

- Method: `POST`
- URL: `https://api.acta.build/credentials`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...OWNER_PUBLIC_KEY...",
  "vaultContractId": "C...VAULT_CONTRACT_ADDRESS...",
  "vcId": "vc:example:123",
  "vcData": "{\"type\":\"Attestation\",\"subject\":\"...\"}",
  "didUri": "did:acta:owner-123"
}
```

Success (201):
```json
{ "vc_id": "vc:example:123", "tx_id": "<transaction-hash>" }
```

Errors:
- 400 `bad_request` — validation or simulation error
- 403 `issuer_not_authorized` — issuer not authorized in owner’s Vault
- 500 `issue_error` — internal processing error