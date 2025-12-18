# Issuance Version

Get the version of the Issuance contract.

## GET `/issuance/version`

Retrieve the contract version. This is a read-only operation that does not require signing.

### Query Parameters

- `issuanceContractId` (optional, string): Issuance contract ID (C...). Uses default contract if not provided

### Request Example

```
GET /issuance/version?issuanceContractId=CDNODJZ6WVTAJHY2DVVQSCGV35TR3276L4S4NLMR33X5NTLASC4K62L7
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
