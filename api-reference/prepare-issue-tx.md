# POST /tx/prepare/issue

Prepares an unsigned XDR to issue a VC via the Issuance contract. Intended for client‑signed flows.

- Method: `POST`
- URL: `https://api.acta.build/tx/prepare/issue`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...OWNER_PUBLIC_KEY...",
  "vcId": "vc:example:123",
  "vcData": "{...W3C VC JSON...}",
  "vaultContractId": "C..." , // optional
  "issuer": "G...",            // optional (signer); defaults to owner
  "issuerDid": "did:pkh:stellar:testnet:G..." // optional
}
```

Response (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errors:
- 400 `owner_required` | `vcId_required` | `vcData_required` — missing fields
- 400 `bad_request` — simulation/validation error details
- 500 `prepare_error` — internal error

Usage notes:
- If `issuer` is provided and valid, the transaction source is `issuer`; otherwise it is `owner`.
- `issuerDid` defaults to `did:pkh:stellar:testnet:<signer>` when omitted.
- After preparing, sign `unsignedXdr` with the corresponding wallet and submit via your client flow.