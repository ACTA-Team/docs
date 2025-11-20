# Payloads

Request body schemas per endpoint (from controllers).

- POST `/credentials` — Signed XDR flow (`api/testnet/src/controllers/credentialsController.ts:12-33`)
  - `{ signedXdr: string; vcId: string }`

- POST `/credentials` — Server-issued flow (`api/testnet/src/controllers/credentialsController.ts:36-64`)
  - `{ owner: string; vcId: string; vcData: string; vaultContractId: string; didUri?: string }`

- POST `/tx/prepare/store` (`api/testnet/src/controllers/txPrepareStoreController.ts:5-37`)
  - `{ owner: string; vcId: string; didUri: string; fields: Record<string, unknown>; vaultContractId?: string; issuer?: string }`

- POST `/tx/prepare/issue` (`api/testnet/src/controllers/txPrepareIssueController.ts:5-20`)
  - `{ owner: string; vcId: string; vcData: string; vaultContractId?: string }`

- POST `/tx/prepare/list_vc_ids` (`api/testnet/src/controllers/txPrepareListVcIdsController.ts:5-13`)
  - `{ owner: string; vaultContractId?: string }`

- POST `/tx/prepare/get_vc` (`api/testnet/src/controllers/txPrepareGetVcController.ts:5-16`)
  - `{ owner: string; vcId: string; vaultContractId?: string }`

- POST `/vault/store` — Signed XDR flow (`api/testnet/src/controllers/vaultStoreController.ts:13-28`)
  - `{ signedXdr: string; vcId: string; owner?: string; vaultContractId?: string }`

- POST `/vault/store` — Server-side flow (`api/testnet/src/controllers/vaultStoreController.ts:46-73`)
  - `{ owner: string; vcId: string; vcData: string; vaultContractId: string; issuerDid?: string; issuanceContractId?: string }`

- POST `/vault/list_vc_ids` (`api/testnet/src/controllers/vaultListVcIdsController.ts:6-21`)
  - Either `{ signedXdr: string }` or `{ owner: string; vaultContractId?: string }`

- POST `/vault/get_vc` (`api/testnet/src/controllers/vaultGetVcController.ts:6-22`)
  - Either `{ signedXdr: string }` or `{ owner: string; vcId: string; vaultContractId?: string }`

- POST `/vault/revoke_issuer` (`api/testnet/src/controllers/vaultRevokeIssuerController.ts:4-31`)
  - Either `{ signedXdr: string }` or `{ owner: string; issuer: string; vaultContractId: string }`

- POST `/verify` (`api/testnet/src/controllers/verifyController.ts:14-36`)
  - Either `{ owner: string; vcId: string; vaultContractId?: string }` or `{ vcId: string }`