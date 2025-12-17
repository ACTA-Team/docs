# Prepare Authorize Issuers

Prepare an unsigned XDR transaction to authorize multiple issuers in a vault.

## POST `/tx/prepare/authorize_issuers`

Prepare a transaction to authorize multiple issuers at once to store credentials in an owner's vault. The transaction must be signed and submitted via `/vault/authorize_issuer` with `signedXdr`.

### Request Body

```json
{
  "owner": "G...",
  "issuers": ["G...", "G...", "G..."],
  "vaultContractId": "C...",
  "sourcePublicKey": "G..."
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `issuers` (required, array of strings): Array of Stellar addresses of issuers to authorize (G...)
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
- `400 issuers_required`: Missing issuers parameter (must be non-empty array)
- `400 sourcePublicKey_required`: Missing sourcePublicKey parameter
- `500 authorize_error`: Error preparing transaction

### Notes

- Only the vault admin (owner by default) can authorize issuers
- All issuers in the array will be authorized in a single transaction
- Each issuer address must be a valid Stellar address (G...)
- After receiving the XDR, the client must sign it and submit it via `/vault/authorize_issuer` with `signedXdr` (uses the same endpoint as single issuer authorization)
