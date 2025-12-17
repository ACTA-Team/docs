# Payloads

Request body schemas per endpoint (from controllers).

- POST `/credentials` — Signed XDR flow (`api/src/controllers/credentialsController.ts`)
  - `{ signedXdr: string; vcId: string }`

- POST `/credentials` — Server-issued flow (`api/src/controllers/credentialsController.ts`)
  - `{ owner: string; vcId: string; vcData: string; vaultContractId: string; didUri?: string }`

- POST `/tx/prepare/store` (`api/src/controllers/txPrepareStoreController.ts`)
  - `{ owner: string; vcId: string; didUri: string; fields: Record<string, unknown>; vaultContractId?: string; issuer?: string }`

- POST `/tx/prepare/issue` (`api/src/controllers/txPrepareIssueController.ts`)
  - `{ owner: string; vcId: string; vcData: string; vaultContractId?: string }`

- POST `/tx/prepare/initialize` (`api/src/controllers/vaultInitializeController.ts`)
  - `{ owner: string; didUri: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/tx/prepare/authorize_issuer` (`api/src/controllers/vaultAuthorizeIssuerController.ts`)
  - `{ owner: string; issuer: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/tx/prepare/authorize_issuers` (`api/src/controllers/vaultAuthorizeIssuersController.ts`)
  - `{ owner: string; issuers: string[]; vaultContractId?: string; sourcePublicKey: string }`

- POST `/tx/prepare/push` (`api/src/controllers/vaultPushController.ts`)
  - `{ fromOwner: string; toOwner: string; vcId: string; issuer: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/tx/prepare/revoke_vault` (`api/src/controllers/vaultRevokeVaultController.ts`)
  - `{ owner: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/vault/store` — Signed XDR flow (`api/src/controllers/vaultStoreController.ts`)
  - `{ signedXdr: string; vcId: string; owner?: string; vaultContractId?: string }`

- POST `/vault/store` — Server-side flow (`api/src/controllers/vaultStoreController.ts`)
  - `{ owner: string; vcId: string; vcData: string; vaultContractId: string; issuerDid?: string; issuanceContractId?: string }`

- POST `/vault/list_vc_ids` (`api/src/controllers/vaultListVcIdsController.ts`)
  - `{ owner: string; vaultContractId?: string }` (read-only, no signature required)

- POST `/vault/get_vc` (`api/src/controllers/vaultGetVcController.ts`)
  - `{ owner: string; vcId: string; vaultContractId?: string }` (read-only, no signature required)

- POST `/vault/verify_vc` (`api/src/controllers/vaultVerifyVcController.ts`)
  - `{ owner: string; vcId: string; vaultContractId?: string }` (read-only, no signature required)

- POST `/vault/initialize` (`api/src/controllers/vaultInitializeController.ts`)
  - Either `{ signedXdr: string }` or `{ owner: string; didUri: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/vault/authorize_issuer` (`api/src/controllers/vaultAuthorizeIssuerController.ts`)
  - Either `{ signedXdr: string }` or `{ owner: string; issuer: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/vault/push` (`api/src/controllers/vaultPushController.ts`)
  - Either `{ signedXdr: string }` or `{ fromOwner: string; toOwner: string; vcId: string; issuer: string; vaultContractId?: string; sourcePublicKey: string }`

- POST `/vault/revoke_issuer` (`api/src/controllers/vaultRevokeIssuerController.ts`)
  - Either `{ signedXdr: string }` or `{ owner: string; issuer: string; vaultContractId: string }`

- POST `/vault/revoke_vault` (`api/src/controllers/vaultRevokeVaultController.ts`)
  - `{ owner: string; vaultContractId?: string; sourcePublicKey: string }` (prepare only)

- GET `/vault/version` (`api/src/controllers/vaultVersionController.ts`)
  - Query: `?vaultContractId=C...` (read-only, no signature required)

- GET `/issuance/version` (`api/src/controllers/issuanceVersionController.ts`)
  - Query: `?issuanceContractId=C...` (read-only, no signature required)

- POST `/issuance/revoke` (`api/src/controllers/issuanceRevokeController.ts`)
  - `{ vcId: string; date?: string }`

- GET `/verify/:vc_id` (`api/src/controllers/verifyController.ts`)
  - Path parameter: `vc_id` (read-only, no signature required)

- POST `/verify` (`api/src/controllers/verifyController.ts`)
  - Either `{ owner: string; vcId: string; vaultContractId?: string }` (vault verification) or `{ vcId: string }` (issuance verification) (read-only, no signature required)
