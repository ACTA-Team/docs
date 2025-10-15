# GET /health

Returns service health and environment info.

- Method: `GET`
- URL: `https://api.acta.build/health`

Response (200):
```json
{
  "status": "OK",
  "timestamp": "2025-01-01T00:00:00.000Z",
  "service": "ACTA API",
  "port": 8000,
  "env": {
    "NODE_ENV": "production",
    "ACTA_ISSUANCE_CONTRACT_ID": "<contract-address>",
    "API_PUBLIC_KEY": "<G-address>"
  }
}
```