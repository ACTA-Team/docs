# GET /health

Returns service status and relevant environment information.

- Method: `GET`
- URLs:
  - Testnet: `https://api.testnet.acta.build/health`
  - Mainnet: `https://api.mainnet.acta.build/health`

Response (200):
```json
{
  "status": "OK",
  "timestamp": "2025-01-01T00:00:00.000Z",
  "service": "ACTA API",
  "port": 8000,
  "env": {
    "NODE_ENV": "production",
    "ACTA_ISSUANCE_CONTRACT_ID": "C...",
    "API_PUBLIC_KEY": "G..."
  }
}
```