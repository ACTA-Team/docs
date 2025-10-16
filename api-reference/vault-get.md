# POST /vault/get_vc

Executes a signed read to fetch a specific VC from the owner’s Vault.

- Method: `POST`
- URL: `https://api.acta.build/vault/get_vc`
- Content-Type: `application/json`

Request body:
```json
{
  "signedXdr": "AAAAAgAAAAA...=="
}
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
- 400 `signed_xdr_required` — missing signed envelope
- 500 `read_error` — internal processing error

Usage notes:
- Use `POST /tx/prepare/get_vc` to obtain `unsignedXdr`, sign with the connected wallet, then POST here with `signedXdr`.
- This call requires owner signature to satisfy `require_auth` on the Vault contract.