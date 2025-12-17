# Prepare Initialize

Prepare an unsigned XDR transaction to initialize a vault.

## POST `/tx/prepare/initialize`

Prepare a transaction to initialize a new vault for an owner. The transaction must be signed and submitted via `/vault/initialize`.

### Request Body

```json
{
  "owner": "G...",
  "didUri": "did:pkh:stellar:testnet:G...",
  "vaultContractId": "C...",
  "sourcePublicKey": "G..."
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `didUri` (required, string): DID URI for the owner (must match format `did:pkh:stellar:network:address`)
- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided
- `sourcePublicKey` (required, string): Public key of the account that will sign the transaction (G...)

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
- `400 didUri_required`: Missing didUri parameter
- `400 didUri_invalid`: Invalid DID URI format
- `400 sourcePublicKey_required`: Missing sourcePublicKey parameter
- `500 initialize_error`: Error preparing transaction

### Notes

- The vault can only be initialized once per owner
- The `didUri` must follow the DID format: `did:pkh:stellar:{network}:{address}`
- After receiving the XDR, the client must sign it and submit it via `/vault/initialize` with `signedXdr`
