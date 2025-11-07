# POST /tx/prepare/get_vc

Prepares an unsigned XDR to fetch a specific VC from the Vault.

- Method: `POST`
- URL: `https://api.acta.build/tx/prepare/get_vc`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...",
  "vcId": "vc:example:123",
  "vaultContractId": "C..." // optional
}
```

Success (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errors:
- 400 `owner_required` | `vcId_required` — missing required fields.
- 400 `bad_request` — simulation error details.
- 500 `prepare_error` — internal error.

Usage notes:
- The server uses `ACTA_VAULT_CONTRACT_ID` when `vaultContractId` is omitted.
- Sign the `unsignedXdr` and submit to `POST /vault/get_vc` as `signedXdr`.