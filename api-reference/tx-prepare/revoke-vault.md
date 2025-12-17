# Prepare Revoke Vault

Prepare an unsigned XDR transaction to revoke a vault.

## POST `/tx/prepare/revoke_vault`

Prepare a transaction to revoke an entire vault, disabling all operations on it. The transaction must be signed and submitted via `/vault/revoke_vault`.

### Request Body

```json
{
  "owner": "G...",
  "vaultContractId": "C...",
  "sourcePublicKey": "G..."
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided
- `sourcePublicKey` (required, string): Public key of the account that will sign the transaction (must be the vault admin, G...)

### Response

**200 OK**

```json
{
  "xdr": "AAAA...",
  "network": "Test SDF Network ; September 2015"
}
```

### Errors

- `400 bad_request`: Invalid parameters or simulation error
- `400 owner_required`: Missing owner parameter
- `400 sourcePublicKey_required`: Missing sourcePublicKey parameter
- `500 revoke_error`: Error preparing transaction

### Notes

- Only the vault admin (owner by default) can revoke the vault
- Once revoked, the vault cannot be used for any operations:
  - Cannot store new credentials
  - Cannot authorize new issuers
  - Cannot perform push operations
  - Existing credentials remain stored but the vault is inactive
- After receiving the XDR, the client must sign it and submit it via `/vault/revoke_vault` with `signedXdr`
