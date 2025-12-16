# Vault Authorize Issuer

Authorize a single issuer to store credentials in an owner's vault.

## POST `/vault/authorize_issuer`

Authorize an issuer with a signed XDR transaction.

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
- `500 authorize_error`: Error authorizing issuer

---

## POST `/tx/prepare/authorize_issuer`

Prepare an unsigned XDR transaction to authorize an issuer.

### Request Body

```json
{
  "owner": "G...",
  "issuer": "G...",
  "vaultContractId": "C...", // optional, uses default if not provided
  "sourcePublicKey": "G..."
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `issuer` (required, string): Stellar address of the issuer to authorize (G...)
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
- `400 issuer_required`: Missing issuer parameter
- `400 sourcePublicKey_required`: Missing sourcePublicKey parameter
- `500 authorize_error`: Error preparing transaction

### Notes

- Only the vault admin (owner by default) can authorize issuers
- The issuer must be authorized before they can store credentials in the vault
- After receiving the XDR, the client must sign it and submit it via `/vault/authorize_issuer` with `signedXdr`

