# GET /verify/{vc_id}

Verifies a credential’s status using the Issuance contract.

- Method: `GET`
- URLs:
  - Testnet: `https://api.testnet.acta.build/verify/{vc_id}`
  - Mainnet: `https://api.mainnet.acta.build/verify/{vc_id}`

Success (200):
```json
{
  "vc_id": "vc:example:123",
  "status": "valid",
  "since": "2025-01-01T00:00:00.000Z"
}
```

Notes:
- `status`: `valid`, `invalid`, or `revoked`.
- `since`: timestamp when revoked (may be null or absent).