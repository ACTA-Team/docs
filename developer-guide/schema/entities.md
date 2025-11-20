# Entities

Credential and verification-related entities returned by the API.

- Credential (Vault read)
  - `vc`: unknown | object — credential contents returned by Vault (`api/testnet/src/controllers/vaultGetVcController.ts:11-22`)

- Verification
  - `vc_id`: string
  - `status`: string
  - `since?`: string
  - Source: `api/testnet/src/services/verify.ts` (via controller) and `api/testnet/src/controllers/verifyController.ts:5-36`

- List of VC IDs
  - `vc_ids`: string[] — list of credential identifiers (`api/testnet/src/controllers/vaultListVcIdsController.ts:11-21`)