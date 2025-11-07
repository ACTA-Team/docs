# POST /credentials

Issue a credential via the Issuance contract and store it in the owner’s Vault.

- Method: `POST`
- URLs:
  - Testnet: `https://api.testnet.acta.build/credentials`
  - Mainnet: `https://api.mainnet.acta.build/credentials`
- Content-Type: `application/json`

Request body:
```json
{
  "owner": "G...",
  "vaultContractId": "C...",
  "vcId": "vc:example:123",
  "vcData": "{\"@context\":[\"https://www.w3.org/2018/credentials/v1\"],\"type\":[\"VerifiableCredential\"],\"credentialSubject\":{...}}",
  "didUri": "did:acta:owner-123"
}
```

Success (201):
```json
{ "vc_id": "vc:example:123", "tx_id": "<transaction-hash>" }
```

Errors:
- 400 `bad_request` — validation or simulation error.
- 403 `issuer_not_authorized` — API issuer is not authorized in the owner’s Vault.
- 500 `issue_error` — internal error.