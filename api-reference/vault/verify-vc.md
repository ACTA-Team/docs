# Vault Verify VC

Verify the status of a credential stored in a vault. This is a read-only operation that does not require signing.

## POST `/vault/verify_vc`

Verify a credential's status by checking with the issuance contract via the vault.

### Request Body

```json
{
  "owner": "G...",
  "vcId": "credential-id-123",
  "vaultContractId": "C..." // optional, uses default if not provided
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `vcId` (required, string): ID of the credential to verify
- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided

### Response

**200 OK**

```json
{
  "status": "valid",
  "since": "2024-01-15T10:30:00Z"
}
```

Or for revoked credentials:

```json
{
  "status": "revoked",
  "since": "2024-01-20T14:00:00Z"
}
```

### Status Values

- `valid`: Credential is valid and not revoked
- `revoked`: Credential has been revoked
- `invalid`: Credential does not exist or is invalid

### Errors

- `400 bad_request`: Invalid parameters or simulation error
- `400 owner_required`: Missing owner parameter
- `400 vcId_required`: Missing vcId parameter
- `500 verify_error`: Error verifying credential

### Notes

- This is a read-only operation (no signature required)
- Uses the vault's `verify_vc` function which delegates to the issuance contract
- Different from `/verify` which uses the issuance contract directly
- The `since` field is only present when the status is `revoked`

