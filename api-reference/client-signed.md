# Client-signed flow (optional)

For workflows where the client signs Soroban transactions, the API provides XDR preparation and submission endpoints.

- `POST /tx/prepare/issue` — prepare XDR to issue a VC
- `POST /tx/prepare/store` — prepare XDR to store a VC
- `POST /tx/submit` — submit a signed XDR

Important security note:
- Your `STELLAR_SECRET_KEY` lives only within your infrastructure (e.g., your `.env`, HSM, or wallet solution).
- The ACTA API never requests, receives, or stores your private keys.

## Prepare Issue XDR

Request body:
```json
{
  "owner": "G...",
  "vaultContractId": "C...",
  "vcId": "vc:example:123",
  "vcData": "{...}",
  "signerPublicKey": "G..."
}
```

Response:
```json
{ "xdr": "AAAA...", "networkPassphrase": "Test SDF Network ; September 2015" }
```

## Prepare Store XDR

Same structure as issue; returns `{ xdr, networkPassphrase }`.

## Submit Signed XDR

Request body:
```json
{ "xdr": "AAAA..." }
```

Response:
```json
{ "tx_id": "<transaction-hash>" }
```

Errors:
- 403 `issuer_not_authorized` — issuer not authorized in owner’s Vault
- 500 `submit_error` — internal error during submission