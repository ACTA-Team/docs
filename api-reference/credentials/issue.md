# Issue Credential

Endpoint to create/issue a credential.

## POST `/credentials`

### Signed XDR flow

- Body: `{ signedXdr: string; vcId: string }`
- Responses:
  - `201`: `{ vc_id: string; tx_id: string }`
  - `400`: `{ error: 'bad_request', message: 'Invalid signedXdr' }`
  - `403`: `{ error: 'issuer_not_authorized' }`
  - `500`: `{ error: 'issue_error', message }`
- Source: `api/testnet/src/controllers/credentialsController.ts:12-33`

### Server-issued flow

- Body: `{ owner: string; vcId: string; vcData: string; vaultContractId: string; didUri?: string }`
- Validations: `owner_required`, `vcId_required`, `vcData_required`, `vaultContractId_required`, `didUri_invalid`
- Response:
  - `201`: `{ vc_id: string; tx_id: string }`
  - Errors: `403 issuer_not_authorized`, `400 bad_request (Simulation error)`, `500 issue_error`
- Source: `api/testnet/src/controllers/credentialsController.ts:36-64`