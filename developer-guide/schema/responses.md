# Responses

Response body schemas per endpoint.

- POST `/credentials` (`api/testnet/src/controllers/credentialsController.ts:12-33,48-64`)
  - `201 { vc_id: string; tx_id: string }`

- POST `/tx/prepare/*` (store/issue/list_vc_ids/get_vc) (`api/testnet/src/controllers/txPrepareStoreController.ts:38-45`, `txPrepareIssueController.ts:19-27`, `txPrepareListVcIdsController.ts:12-20`, `txPrepareGetVcController.ts:15-23`)
  - `200 { unsignedXdr: string }`

- POST `/vault/store` (`api/testnet/src/controllers/vaultStoreController.ts:27-43,83-99`)
  - `201 { vc_id: string; tx_id: string; verification?: { status?: string; since?: string } }`

- POST `/vault/list_vc_ids` (`api/testnet/src/controllers/vaultListVcIdsController.ts:11-21`)
  - `200 { vc_ids: string[] }`

- POST `/vault/get_vc` (`api/testnet/src/controllers/vaultGetVcController.ts:11-22`)
  - `200 { vc: unknown }`

- POST `/vault/revoke_issuer` (`api/testnet/src/controllers/vaultRevokeIssuerController.ts:10-21,31-41`)
  - `201 { tx_id: string }`

- GET `/verify/:vc_id` (`api/testnet/src/controllers/verifyController.ts:5-13`)
  - `200 { vc_id: string; status: string; since?: string }`

- POST `/verify` (`api/testnet/src/controllers/verifyController.ts:14-36`)
  - `200 { vc_id: string; status: string; since?: string }`