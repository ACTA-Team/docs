# Responses

Response body schemas per endpoint.

- POST `/credentials` (`api/src/controllers/credentialsController.ts`)
  - `201 { vc_id: string; tx_id: string }`

- POST `/tx/prepare/*` (store/issue/initialize/authorize_issuer/authorize_issuers/push/revoke_vault) (`api/src/controllers/txPrepare*Controller.ts`)
  - `200 { xdr: string; network: string }` or `200 { unsignedXdr: string }`

- POST `/vault/store` (`api/src/controllers/vaultStoreController.ts`)
  - `201 { vc_id: string; tx_id: string; verification?: { status?: string; since?: string } }`

- POST `/vault/list_vc_ids` (`api/src/controllers/vaultListVcIdsController.ts`)
  - `200 { vc_ids: string[] }`

- POST `/vault/get_vc` (`api/src/controllers/vaultGetVcController.ts`)
  - `200 { vc: unknown }`

- POST `/vault/verify_vc` (`api/src/controllers/vaultVerifyVcController.ts`)
  - `200 { status: string; since?: string }`

- POST `/vault/initialize` (`api/src/controllers/vaultInitializeController.ts`)
  - `201 { tx_id: string }` (signed XDR) or `200 { xdr: string; network: string }` (prepare)

- POST `/vault/authorize_issuer` (`api/src/controllers/vaultAuthorizeIssuerController.ts`)
  - `201 { tx_id: string }` (signed XDR) or `200 { xdr: string; network: string }` (prepare)

- POST `/vault/push` (`api/src/controllers/vaultPushController.ts`)
  - `201 { tx_id: string }` (signed XDR) or `200 { xdr: string; network: string }` (prepare)

- POST `/vault/revoke_issuer` (`api/src/controllers/vaultRevokeIssuerController.ts`)
  - `201 { tx_id: string }`

- POST `/vault/revoke_vault` (`api/src/controllers/vaultRevokeVaultController.ts`)
  - `200 { xdr: string; network: string }` (prepare only)

- GET `/vault/version` (`api/src/controllers/vaultVersionController.ts`)
  - `200 { version: string }`

- GET `/issuance/version` (`api/src/controllers/issuanceVersionController.ts`)
  - `200 { version: string }`

- POST `/issuance/revoke` (`api/src/controllers/issuanceRevokeController.ts`)
  - `201 { vc_id: string; tx_id: string }`

- GET `/verify/:vc_id` (`api/src/controllers/verifyController.ts`)
  - `200 { vc_id: string; status: string; since?: string }`

- POST `/verify` (`api/src/controllers/verifyController.ts`)
  - `200 { vc_id: string; status: string; since?: string }`
