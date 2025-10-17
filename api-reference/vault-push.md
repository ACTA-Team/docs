# POST /vault/push

Moves a VC from one owner to another (client-signed). The request expects a transaction signed by the source owner (`fromOwner`).

- Method: `POST`
- URL: `https://api.acta.build/vault/push`
- Content-Type: `application/json`

Request body:
```json
{
  "signedXdr": "AAAAAgAAAAA...==",
  "vcId": "urn:vc:example:123" // optional echo for client tracking
}
```

Success (201):
```json
{ "vc_id": "urn:vc:example:123", "tx_id": "<transaction-hash>" }
```

Errors:
- 403 `issuer_not_authorized` — issuer not authorized in source Vault (`Error(Contract, #2)`)
- 403 `vault_revoked` — source Vault revoked (`Error(Contract, #4)`)
- 404 `vc_not_found` — VC does not exist in source Vault (`Error(Contract, #6)`)
- 400 `bad_request` — invalid `signedXdr` or simulation error
- 500 `push_error` — internal processing error

Notes:
- `push` requires only the source owner’s signature; the issuer must be authorized but does not sign.
- If your client needs to prepare the unsigned transaction, use `POST /tx/prepare/push`.