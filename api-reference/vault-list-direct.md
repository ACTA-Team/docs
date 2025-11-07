# POST /vault/list_vc_ids_direct

Lists all VC IDs for a Vault without requiring a signed transaction.

- Method: `POST`
- URLs:
  - Testnet: `https://api.testnet.acta.build/vault/list_vc_ids_direct`
  - Mainnet: `https://api.mainnet.acta.build/vault/list_vc_ids_direct`
- Content-Type: `application/json`

Request body:
```json
{ "owner": "G...OWNER_PUBLIC_KEY...", "vaultContractId": "C..." }
```

Success (200):
```json
{ "vc_ids": ["vc:example:acta", "http://university.example/credentials/3732"] }
```

Errors:
- 400 `owner_required` — missing `owner`.
- 404 `vault_not_found` — the Vault for `owner` does not exist.
- 500 `read_error` — internal processing error.

Notes:
- `vaultContractId` is optional; the API uses its configured Vault when omitted.
- Direct endpoints are optimized for read‑only operations where the API can safely resolve state without signatures.
- For wallet‑auth reads, prefer `POST /vault/list_vc_ids` which uses `signedXdr`.