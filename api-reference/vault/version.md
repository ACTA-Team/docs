# Vault Version

Get the version of the Vault contract.

## GET `/vault/version`

Retrieve the contract version. This is a read-only operation that does not require signing.

### Query Parameters

- `vaultContractId` (optional, string): Vault contract ID (C...). Uses default contract if not provided

### Request Example

```
GET /vault/version?vaultContractId=CDDSCINR7TJABVROJFLSZWOFCGXVZWQQUF22C6FIHUQ2JTSTBJNV7ABO
```

### Response

**200 OK**

```json
{
  "version": "1.0.0"
}
```

### Errors

- `400 bad_request`: Invalid parameters or simulation error
- `500 version_error`: Error retrieving version

### Notes

- This is a read-only operation (no signature required)
- Uses transaction simulation to read data from the blockchain
- No transaction is submitted to the network
