# POST /vault/get_vc_direct

Fetches a specific VC from a Vault without requiring a signed transaction.

- Method: `POST`
- URL: `https://api.acta.build/vault/get_vc_direct`
- Content-Type: `application/json`

Request body:
```json
{ "owner": "G...OWNER_PUBLIC_KEY...", "vcId": "vc:example:123", "vaultContractId": "C..." }
```

Success (200):
```json
{
  "vc": {
    "id": "vc:example:123",
    "issuer_did": "did:pkh:stellar:testnet:GABCDE...",
    "issuance_contract": "CANYEUDJCAPQ5AC...",
    "data": "{...}"
  }
}
```

Errors:
- 400 `owner_required` | `vcId_required` — missing fields.
- 404 `vc_not_found` — VC not present in the owner’s Vault.
- 500 `read_error` — internal processing error.

Notes:
- `vaultContractId` is optional; the API uses its configured Vault when omitted.
- For wallet‑auth reads, use `POST /vault/get_vc` with a `signedXdr`.