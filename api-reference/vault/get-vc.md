# Vault Get VC

Get a credential from Vault by ID. This is a read-only operation that does not require signing.

## POST `/vault/get_vc`

Read a credential from the owner's vault.

### Request Body

```json
{
  "owner": "G...",
  "vcId": "credential-id-123",
  "vaultContractId": "C..." // optional, uses default if not provided
}
```

### Parameters

- `owner` (required, string): Stellar address of the vault owner (G...)
- `vcId` (required, string): ID of the credential to retrieve
- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided

### Response

**200 OK**

```json
{
  "vc": {
    // Verifiable Credential object
  }
}
```

### Errors

- `400 bad_request`: Invalid parameters or simulation error
- `400 owner_required`: Missing owner parameter
- `400 vcId_required`: Missing vcId parameter
- `500 read_error`: Error reading credential

### Notes

- This is a read-only operation (no signature required)
- Uses transaction simulation to read data from the blockchain
- No transaction is submitted to the network
