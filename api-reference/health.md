# Health

Service health endpoint.

## GET `/health`

- Returns: `{ status, timestamp, service, port, env: { NODE_ENV, PORT, ACTA_ISSUANCE_CONTRACT_ID, API_PUBLIC_KEY } }`
- Sources: `api/testnet/src/controllers/healthController.ts:5-9`, `api/testnet/src/services/health.ts:3-21`