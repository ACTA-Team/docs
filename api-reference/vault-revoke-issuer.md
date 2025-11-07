# POST /vault/revoke_issuer

Revokes an authorized issuer in a Vault. Supports two modes:

- Method: `POST`
- URLs:
  - Testnet: `https://api.testnet.acta.build/vault/revoke_issuer`
  - Mainnet: `https://api.mainnet.acta.build/vault/revoke_issuer`
- Content-Type: `application/json`

Mode A — Submit signed XDR:
```json
{ "signedXdr": "AAAAAgAAAAA...==" }
```
Success (201):
```json
{ "tx_id": "<transaction-hash>" }
```
Errors:
- 400 `bad_request` — invalid `signedXdr`.
- 500 `revoke_error` — internal error.

Mode B — Direct (server signs, for testing/special cases):
```json
{ "owner": "G...", "issuer": "G...", "vaultContractId": "C..." }
```
Success (201):
```json
{ "tx_id": "<transaction-hash>" }
```
Errors:
- 400 `owner_required` | `issuer_required` | `vaultContractId_required` — missing fields.
- 500 `revoke_error` — internal error.

Notes:
- The direct mode requires the API to hold a signing key and is not typical for production.
- For wallet‑signed flows, prepare and sign the transaction client‑side, then use Mode A.