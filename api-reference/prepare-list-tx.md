# POST /tx/prepare/list_vc_ids

Prepares an unsigned XDR to list VC IDs from the Vault.

- Method: `POST`
- URLs:
  - Testnet: `https://api.testnet.acta.build/tx/prepare/list_vc_ids`
  - Mainnet: `https://api.mainnet.acta.build/tx/prepare/list_vc_ids`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...",
  "vaultContractId": "C..." // optional
}
```

Success (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errors:
- 400 `owner_required` — missing `owner`.
- 400 `bad_request` — simulation error details.
- 500 `prepare_error` — internal error.

Usage notes:
- The server uses `ACTA_VAULT_CONTRACT_ID` when `vaultContractId` is omitted.
- Sign the `unsignedXdr` and submit to `POST /vault/list_vc_ids` as `signedXdr`.