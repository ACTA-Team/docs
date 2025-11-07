# GET /config

Returns the active API configuration useful for clients integrating with ACTA.

- Method: `GET`
- URLs:
  - Testnet: `https://api.testnet.acta.build/config`
  - Mainnet: `https://api.mainnet.acta.build/config`

Success (200):
```json
{
  "rpcUrl": "https://horizon-testnet.stellar.org",
  "networkPassphrase": "Test SDF Network ; September 2015",
  "issuanceContractId": "C...",
  "vaultContractId": "C..."
}
```

Notes:
- Values reflect the environment where the API is running (e.g., testnet vs mainnet).
- Use `issuanceContractId` and `vaultContractId` to target the correct Soroban contracts.