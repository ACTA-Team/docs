# Issuance Revoke

Revoke a credential that was issued through the Issuance contract.

## POST `/issuance/revoke`

Revoke a credential with a signed XDR transaction.

### Request Body (Signed XDR)

```json
{
  "signedXdr": "AAAA..."
}
```

### Response (Signed XDR)

**201 Created**

```json
{
  "tx_id": "transaction-hash"
}
```

### Errors (Signed XDR)

- `400 bad_request`: Invalid signedXdr or simulation error
- `500 revoke_error`: Error revoking credential

---

## Direct Revoke

Revoke a credential directly using server-side signing.

### Request Body

```json
{
  "vcId": "credential-id-123",
  "date": "2024-01-20T14:00:00Z"
}
```

### Parameters

- `vcId` (required, string): ID of the credential to revoke
- `date` (optional, string): ISO 8601 date string for the revocation date. If not provided, current date is used

### Response

**201 Created**

```json
{
  "tx_id": "transaction-hash"
}
```

### Errors

- `400 vcId_required`: Missing vcId parameter
- `500 revoke_error`: Error revoking credential

### Notes

- Requires `STELLAR_SECRET_KEY` to be configured on the server
- Only the credential owner or the contract admin can revoke a credential
- Once revoked, the credential status will be "revoked" when verified
- The revocation date is stored and returned in verification responses

