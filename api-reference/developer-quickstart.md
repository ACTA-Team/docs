# Developer Quickstart

Run the API locally.

## Steps

```bash
cd api/testnet
npm install
npm run dev
```

- Starts on `http://localhost:8000` (`api/testnet/src/index.ts:18-23,75-81`).
- `GET /` returns index JSON with available endpoints (`api/testnet/src/index.ts:26-47`).

## Headers

- CORS allows standard headers and `X-ACTA-Key` (`api/testnet/src/config/cors.ts:4-23`).