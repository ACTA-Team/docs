# POST /tx/prepare/get_vc (Undocumented in Swagger)

Prepares an unsigned XDR to fetch a specific VC from the Vault. This endpoint is intended for client-signed flows and is not exposed in Swagger.

- Method: `POST`
- URL: `https://api.acta.build/tx/prepare/get_vc`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...OWNER_PUBLIC_KEY...",
  "vcId": "vc:example:123",
  "vaultContractId": "C...VAULT_CONTRACT_ADDRESS (optional)"
}
```

Response (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errors:
- 400 `owner_required` | `vcId_required` — missing required fields
- 400 `bad_request` — simulation error details
- 500 `prepare_error` — internal error

Usage notes:
- The server reads `ACTA_VAULT_CONTRACT_ID` from its environment when `vaultContractId` is not provided.
- The client should sign `unsignedXdr` with the connected wallet and then submit it to `POST /vault/get_vc` with `signedXdr`.