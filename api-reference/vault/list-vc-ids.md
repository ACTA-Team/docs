# Vault List VC IDs

List all credential IDs stored in a vault. This is a read-only operation that does not require signing.

## POST `/vault/list_vc_ids`

List all credential IDs for an owner's vault.

### Request Body

```json
{
  "owner": "G...",
  "vaultContractId": "C..." // optional, uses default if not provided
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided

### Response

**200 OK**

```json
{
  "vc_ids": [
    "credential-id-1",
    "credential-id-2",
    "credential-id-3"
  ]
}
```

### Errors

- `400 bad_request`: Invalid parameters or simulation error
- `400 owner_required`: Missing owner parameter
- `500 read_error`: Error reading credential IDs
- `500 unexpected_return_shape`: Invalid response format

### Notes

- This is a read-only operation (no signature required)
- Uses transaction simulation to read data from the blockchain
- No transaction is submitted to the network
- Returns an empty array if the vault has no credentials
