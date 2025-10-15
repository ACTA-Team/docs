# GET /verify/{vc_id}

Verifies the status of a credential via the issuance contract.

- Method: `GET`
- URL: `https://api.acta.build/verify/{vc_id}`

Success (200):
```json
{
  "vc_id": "vc:example:123",
  "status": "valid",
  "since": "2025-01-01T00:00:00.000Z"
}
```

Notes:
- `status` may be `valid`, `invalid`, or `revoked`.
- `since` may be `null` or missing if not applicable.