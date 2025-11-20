# Prepare Issue Transaction

Build unsigned XDR to issue a VC via the Issuance contract.

## POST `/tx/prepare/issue`

- Body: `{ owner: string; vcId: string; vcData: string; vaultContractId?: string }`
- Returns: `{ unsignedXdr: string }`
- Validation errors: `owner_required`, `vcId_required`, `vcData_required`, `bad_request` (simulation)
- Source: `api/testnet/src/controllers/txPrepareIssueController.ts:5-27`