# Vault Push

Move a credential from one vault to another. The credential is transferred from the source owner's vault to the destination owner's vault.

## POST `/vault/push`

Move a credential with a signed XDR transaction.

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
- `500 push_error`: Error moving credential

---

## POST `/tx/prepare/push`

Prepare an unsigned XDR transaction to move a credential between vaults.

### Request Body

```json
{
  "fromOwner": "G...",
  "toOwner": "G...",
  "vcId": "credential-id-123",
  "issuer": "G...",
  "vaultContractId": "C...", // optional, uses default if not provided
  "sourcePublicKey": "G..."
}
```

### Parameters

- `fromOwner` (required, string): Stellar address of the source vault owner (G...)
- `toOwner` (required, string): Stellar address of the destination vault owner (G...)
- `vcId` (required, string): ID of the credential to move
- `issuer` (required, string): Stellar address of the issuer (must be authorized in source vault, G...)
- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided
- `sourcePublicKey` (required, string): Public key of the account that will sign the transaction (must be fromOwner, G...)

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
- `400 fromOwner_required`: Missing fromOwner parameter
- `400 toOwner_required`: Missing toOwner parameter
- `400 vcId_required`: Missing vcId parameter
- `400 issuer_required`: Missing issuer parameter
- `400 sourcePublicKey_required`: Missing sourcePublicKey parameter
- `500 push_error`: Error preparing transaction

### Notes

- The `fromOwner` must sign the transaction
- The `issuer` must be authorized in the source vault
- Both source and destination vaults must be active (not revoked)
- The credential is removed from the source vault and added to the destination vault
- After receiving the XDR, the client must sign it and submit it via `/vault/push` with `signedXdr`

