# Vault Revoke Issuer

Revoke authorization for an issuer in a vault. Once revoked, the issuer can no longer store credentials in the vault.

## POST `/vault/revoke_issuer`

Revoke issuer authorization with a signed XDR transaction or direct execution.

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
- `500 revoke_error`: Error revoking issuer authorization

---

### Request Body (Direct)

```json
{
  "owner": "G...",
  "issuer": "G...",
  "vaultContractId": "C..."
}
```

### Parameters (Direct)

- `owner` (required, string): Stellar address of the vault owner (G...)
- `issuer` (required, string): Stellar address of the issuer to revoke (G...)
- `vaultContractId` (required, string): Vault contract ID (C...)

### Response (Direct)

**201 Created**

```json
{
  "tx_id": "transaction-hash"
}
```

### Errors (Direct)

- `400 bad_request`: Invalid parameters or simulation error
- `400 owner_required`: Missing owner parameter
- `400 issuer_required`: Missing issuer parameter
- `400 vaultContractId_required`: Missing vaultContractId parameter
- `500 revoke_error`: Error revoking issuer authorization

### Notes

- Only the vault admin (owner by default) can revoke issuer authorization
- Once revoked, the issuer cannot store new credentials in the vault
- Existing credentials stored by the issuer remain in the vault
- Direct flow requires `STELLAR_SECRET_KEY` to be configured on the server
