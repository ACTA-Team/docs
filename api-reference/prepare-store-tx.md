# POST /tx/prepare/store (Undocumented in Swagger)

Prepares an unsigned XDR for `store_vc` using normal form fields. This endpoint is intended for client-signed flows and is not exposed in Swagger.

- Method: `POST`
- URL: `https://api.acta.build/tx/prepare/store`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...OWNER_PUBLIC_KEY...",
  "vcId": "vc:example:123",
  "didUri": "did:pkh:stellar:testnet:G...",
  "fields": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "type": "Attestation",
    "expires": "2025-12-31"
  }
}
```

Response (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errors:
- 400 `owner_required` | `vcId_required` | `didUri_required` — missing required fields
- 400 `bad_request` — simulation error details
- 500 `prepare_error` — internal error

Usage notes:
- The server reads `ACTA_VAULT_CONTRACT_ID` from its environment.
- The client should sign `unsignedXdr` with the connected wallet and then submit it to `POST /vault/store` with `signedXdr`.