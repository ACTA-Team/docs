# Verify Credential (GET)

Verify credential status via the Issuance contract.

## GET `/verify/:vc_id`

Verify a credential's status by checking with the Issuance contract.

### Path Parameters

- `vc_id` (required, string): ID of the credential to verify

### Request Example

```
GET /verify/credential-id-123
```

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

- `400 vc_id_required`: Missing vc_id in path
- `500 verify_error`: Error verifying credential

### Notes

- This is a read-only operation (no signature required)
- Uses the Issuance contract's `verify` function directly
- Different from `/vault/verify_vc` which verifies via the vault
- The `since` field is only present when the status is `revoked`
