# POST /vault/store

Store a credential directly in the Vault. The API signs the transaction (server‑signed). The default issuer is the API public key (`API_PUBLIC_KEY`).

- Method: `POST`
- URL: `https://api.acta.build/vault/store`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...",
  "vaultContractId": "C...",
  "vcId": "vc:example:123",
  "vcData": "{\"type\":\"Attestation\",\"subject\":\"...\"}",
  "issuerDid": "did:pkh:stellar:testnet:G...",
  "issuanceContractId": "C..." // optional, if overriding
}
```

Success (201):
```json
{ "vc_id": "vc:example:123", "tx_id": "<transaction-hash>" }
```

Errors:
- 400 `bad_request` — validation or simulation error.
- 403 `issuer_not_authorized` — API issuer is not authorized in the owner’s Vault.
- 500 `store_error` — internal error.