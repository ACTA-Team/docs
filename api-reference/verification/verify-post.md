# Verify Credential (POST)

Verify credential status via Vault or Issuance contract depending on the payload.

## POST `/verify`

Unified verification endpoint that can verify credentials either through the Issuance contract or through a Vault.

### Request Body (Vault Verification)

If `owner` is provided, verifies via the Vault contract:

```json
{
  "owner": "G...",
  "vcId": "credential-id-123",
  "vaultContractId": "C..." // optional, uses default if not provided
}
```

### Request Body (Issuance Verification)

If `owner` is not provided, verifies via the Issuance contract:

```json
{
  "vcId": "credential-id-123"
}
```

Or:

```json
{
  "vc_id": "credential-id-123"
}
```

### Parameters

- `owner` (optional, string): Stellar address of the vault owner (G...). If provided, uses vault verification
- `vcId` or `vc_id` (required, string): ID of the credential to verify
- `vaultContractId` (optional, string): Vault contract ID (C...). Only used if `owner` is provided

### Response

**200 OK**

```json
{
  "vc_id": "credential-id-123",
  "status": "valid",
  "since": "2024-01-20T14:00:00Z"
}
```

Or for revoked credentials:

```json
{
  "vc_id": "credential-id-123",
  "status": "revoked",
  "since": "2024-01-20T14:00:00Z"
}
```

### Status Values

- `valid`: Credential is valid and not revoked
- `revoked`: Credential has been revoked
- `invalid`: Credential does not exist or is invalid

### Errors

- `400 vcId_required`: Missing vcId parameter
- `500 verify_error`: Error verifying credential

### Notes

- This is a read-only operation (no signature required)
- Uses transaction simulation to read data from the blockchain
- **If `owner` is provided**: Uses the Vault contract's `verify_vc` function
- **If `owner` is not provided**: Uses the Issuance contract's `verify` function
- The `since` field is only present when the status is `revoked`
- Equivalent to GET `/verify/:vc_id` when `owner` is not provided
- Equivalent to POST `/vault/verify_vc` when `owner` is provided
