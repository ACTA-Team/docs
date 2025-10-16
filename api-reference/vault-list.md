# POST /vault/list_vc_ids

Executes a signed read to list all VC IDs for the owner’s Vault.

- Method: `POST`
- URL: `https://api.acta.build/vault/list_vc_ids`
- Content-Type: `application/json`

Request body:
```json
{
  "signedXdr": "AAAAAgAAAAA...=="
}
```

Success (200):
```json
{ "vc_ids": ["vc:example:acta", "http://university.example/credentials/3732"] }
```

Errors:
- 400 `signed_xdr_required` — missing signed envelope
- 500 `read_error` — internal processing error or unexpected return shape

Usage notes:
- Use `POST /tx/prepare/list_vc_ids` to obtain `unsignedXdr`, sign with the connected wallet, then POST here with `signedXdr`.
- This call requires owner signature to satisfy `require_auth` on the Vault contract.