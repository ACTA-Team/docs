# POST /tx/prepare/push

Prepares an unsigned XDR for `push` to move a VC from one owner to another. This endpoint is intended for client-signed flows.

- Method: `POST`
- URL: `https://api.acta.build/tx/prepare/push`
- Content-Type: `application/json`

Request body:
```json
{
  "fromOwner": "G...SOURCE_OWNER_PUBLIC_KEY...",
  "toOwner": "G...DEST_OWNER_PUBLIC_KEY...",
  "vcId": "urn:vc:example:123",
  "issuer": "G...ISSUER_PUBLIC_KEY...",
  "vaultContractId": "C...VAULT_CONTRACT_ADDRESS..." // optional
}
```

Response (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errors:
- 400 `fromOwner_required` | `toOwner_required` | `vcId_required` | `issuer_required` — missing fields
- 400 `bad_request` — simulation/validation error details
- 500 `prepare_error` — internal error

Usage notes:
- The client must sign `unsignedXdr` using the wallet of `fromOwner` and then submit it to `POST /vault/push`.
- The contract validates that `issuer` is authorized in the source Vault (`fromOwner`). If you receive `issuer_not_authorized`, authorize the issuer and retry.