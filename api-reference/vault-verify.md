# POST /vault/verify

Verifies a credential’s status using data in the owner’s Vault.

- Method: `POST`
- URLs:
  - Testnet: `https://api.testnet.acta.build/vault/verify`
  - Mainnet: `https://api.mainnet.acta.build/vault/verify`
- Content-Type: `application/json`

Request body:
```json
{ "owner": "G...OWNER_PUBLIC_KEY...", "vcId": "vc:example:123", "vaultContractId": "C..." }
```

Success (200):
```json
{ "vc_id": "vc:example:123", "status": "valid", "since": null }
```

Errors:
- 400 `owner_required` | `vcId_required` — missing fields.
- 404 `vc_not_found` — VC not present in the owner’s Vault.
- 500 `verify_error` — internal error.

Notes:
- `status`: `valid`, `invalid`, or `revoked`.
- `since`: timestamp when revoked (if applicable).
- For cross‑owner verification via the Issuance contract, use `GET /verify/{vc_id}`.