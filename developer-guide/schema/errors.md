# Errors

Standardized error codes and validation messages from controllers.

- Validation errors
  - `owner_required`, `issuer_required`, `vcId_required`, `vcData_required`, `vaultContractId_required`, `didUri_required`, `issuerDid_invalid`, `signed_xdr_invalid`
  - Sources: `api/testnet/src/controllers/*.ts`

- Prepare errors
  - `bad_request` (simulation), `prepare_error`
  - Sources: `txPrepareStoreController.ts:41-45`, `txPrepareIssueController.ts:23-27`, `txPrepareListVcIdsController.ts:15-20`, `txPrepareGetVcController.ts:19-23`

- Vault errors
  - `store_error`, `unexpected_return_shape`, `read_error`, `revoke_error`
  - Sources: `vaultStoreController.ts:94-97`, `vaultListVcIdsController.ts:12-14,22-27`, `vaultGetVcController.ts:23-28`, `vaultRevokeIssuerController.ts:20-41`

- Credentials errors
  - `issue_error`, `issuer_not_authorized`, `bad_request`, `signed_xdr_invalid`
  - Sources: `credentialsController.ts:19-33,52-63`

- Verification errors
  - `verify_error`, `vcId_required`
  - Sources: `verifyController.ts:10-13,31-36`