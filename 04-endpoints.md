# API Endpoints

## Overview

The ACTA API provides RESTful endpoints for managing verifiable credentials on the Stellar blockchain. All endpoints follow consistent patterns for authentication, request/response formats, and error handling.

### **Base URLs**

| Environment | URL | Status |
|-------------|-----|--------|
| **Development** | `http://localhost:8000` | Active |
| **Production** | `https://api.acta.build` | Active |

---

## Authentication

API keys are supported but not required. If you use keys.acta.build, you may include `X-ACTA-Key` for tracking, but the current API endpoints do not enforce API key authentication.

**Example Request:**
```bash
curl -X POST https://api.acta.build/credentials \
  -H "Content-Type: application/json" \
  -d '{"data": {...}}'
```

---

## Response Format

All API responses follow a consistent JSON structure for both success and error cases.

### **Success Response**

```json
{
  "success": true,
  "data": {
    // Response data here
  },
  "message": "Operation completed successfully"
}
```

### **Error Response**

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message"
  }
}
```

---

## **Root & Health**

### `GET /`

Serves Swagger UI for interactive documentation. Requests to `/` return the documentation UI (HTML).

---

Removed. The `/ping` endpoint no longer exists.

---

### `GET /health`

Service status.

**Endpoint:** `GET /health`

**Description:** Detailed health check that verifies API and Stellar network status.

**Request:**

```bash
curl -X GET https://api.acta.build/health
```

**Response:**

```json
{
  "status": "OK",
  "timestamp": "2025-01-10T12:00:00Z",
  "service": "Stellar Credential API",
  "port": 8000,
  "env": { "NODE_ENV": "development", "PORT": "8000" }
}
```

**Status Codes:**
- `200 OK`: Service is healthy
- `503 Service Unavailable`: Service is unhealthy

---

## **Credentials**

All credential-related operations for creating, retrieving, and managing verifiable credentials.

### `POST /credentials`

Creates a new Verifiable Credential (VC) on the Stellar blockchain.

**Endpoint:** `POST /credentials`

**Description:** Anchors the VC `sha256` on-chain using Soroban contracts (`issuance.issue`).

**Request:**

```bash
curl -X POST https://api.acta.build/credentials \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "@context": [
        "https://www.w3.org/ns/credentials/v2",
        "https://www.w3.org/ns/credentials/examples/v2"
      ],
      "type": ["VerifiableCredential", "UniversityDegreeCredential"],
      "issuer": "did:example:76e12ec712ebc6f1c221ebfeb1f",
      "issuanceDate": "2024-01-15T10:00:00Z",
      "credentialSubject": {
        "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
        "degree": { "type": "BachelorDegree", "name": "Bachelor of Science" }
      }
    },
    "metadata": {
      "issuer": "University of Example",
      "subject": "John Doe",
      "expirationDate": "2029-01-15T10:00:00Z"
    }
  }'
```

**Request Body Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `data` | Object | Yes | The credential data to be stored on blockchain |
| `metadata` | Object | No | Additional metadata about the credential |
| `metadata.issuer` | String | No | The issuer of the credential |
| `metadata.subject` | String | No | The subject of the credential |
| `metadata.expirationDate` | String | No | ISO 8601 expiration date |

**Response:**

```json
{
  "success": true,
  "data": {
    "contractId": "CXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "hash": "<sha256 of VC>",
    "transactionHash": "<tx hash>",
    "ledgerSequence": 123456,
    "createdAt": "2025-01-10T12:00:00Z"
  }
}
```

**Status Codes:**
- `201 Created`: Credential created successfully
- `400 Bad Request`: Missing required field: data
- `500 Internal Server Error`: Failed to create credential

---

### `GET /credentials/:contractId`

Retrieves a credential by its Stellar contract ID.

**Endpoint:** `GET /credentials/:contractId`

**Description:** Returns on-chain status via `issuance.verify` and the hash if present in account `manageData`.

**Request:**

```bash
curl -X GET https://api.acta.build/credentials/CXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contractId` | String | Yes | The Stellar contract ID of the credential |

**Response:**

```json
{
  "success": true,
  "data": {
    "contractId": "CXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "hash": "<sha256 if present>",
    "status": "Active"
  }
}
```

**Status Codes:**
- `200 OK`: Credential found
- `400 Bad Request`: Contract ID is required
- `500 Internal Server Error`: Failed to get credential

---

### `PATCH /credentials/:contractId/status`

Updates the status of an existing credential.

**Endpoint:** `PATCH /credentials/:contractId/status`

**Description:** Updates the status of a credential (Active, Revoked, or Suspended) on the blockchain.

**Request:**

```bash
curl -X PATCH https://api.acta.build/credentials/CXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/status \
  -H "Content-Type: application/json" \
  -d '{"status": "Revoked"}'
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contractId` | String | Yes | The Stellar contract ID (path parameter) |
| `status` | String | Yes | New status: "Active", "Revoked", or "Suspended" |

**Response:**

```json
{ "success": true, "message": "Credential status updated successfully" }
```

**Status Codes:**
- `200 OK`: Status updated successfully
- `400 Bad Request`: Invalid status or missing contract ID
- `500 Internal Server Error`: Failed to update status

---

### `GET /credentials/hash/:hash`

Removed. Hash lookup will be reintroduced when `VaultContract` exposes getters.

---

### GET /credentials/:contractId

Retrieves information about a specific credential by its contract ID.

**Request:**
```http
GET /credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contractId` | String | Yes | The Stellar contract ID (56 characters, starts with 'C') |

**Response:**
```json
{
  "success": true,
  "data": {
    "contractId": "CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ",
    "hash": "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456",
    "status": "Active"
  }
}
```

**Status Codes:**
- `200 OK`: Credential found
- `400 Bad Request`: Invalid contract ID format
- `404 Not Found`: Credential not found
- `500 Internal Server Error`: Server error

**Example cURL:**
```bash
curl -X GET https://api.acta.build/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ
```

---

### PATCH /credentials/:contractId/status

Updates the status of an existing credential.

**Request:**
```http
PATCH /credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ/status
Content-Type: application/json

{
  "status": "Revoked"
}
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contractId` | String | Yes | The Stellar contract ID |

**Request Body Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | String | Yes | New status: "Active", "Revoked", or "Suspended" |

**Response:**
```json
{
  "success": true,
  "message": "Credential status updated successfully"
}
```

**Status Codes:**
- `200 OK`: Status updated successfully
- `400 Bad Request`: Invalid contract ID or status
- `404 Not Found`: Credential not found
- `500 Internal Server Error`: Server error

**Valid Status Values:**
- `Active`: Credential is valid and active
- `Revoked`: Credential has been permanently revoked
- `Suspended`: Credential is temporarily suspended

**Example cURL:**
```bash
curl -X PATCH https://api.acta.build/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ/status \
  -H "Content-Type: application/json" \
  -d '{"status": "Revoked"}'
```

---

### GET /credentials/hash/:hash

Retrieves credential information by its data hash.

**Request:**
```http
GET /credentials/hash/a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hash` | String | Yes | SHA-256 hash of the credential data (64 hex characters) |

**Response:**
```json
{
  "success": true,
  "data": {
    "contractId": "CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ",
    "hash": "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456",
    "status": "Active",
    "createdAt": "2025-01-15T10:30:00.000Z",
    "transactionHash": "7f8e9d0c1b2a3456789012345678901234567890abcdef1234567890abcdef12",
    "ledgerSequence": 12345678
  }
}
```

**Status Codes:**
- `200 OK`: Credential found
- `400 Bad Request`: Invalid hash format
- `404 Not Found`: Credential not found
- `500 Internal Server Error`: Server error

**Example cURL:**
```bash
curl -X GET https://api.acta.build/credentials/hash/a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456 \
  -H "Authorization: Bearer YOUR_API_KEY"
```

---

## Error Handling

### Common Error Responses

#### 400 Bad Request
```json
{
  "error": "Missing required field: data"
}
```

#### 404 Not Found
```json
{
  "error": "Endpoint not found",
  "path": "/invalid-endpoint",
  "method": "GET"
}
```

#### 500 Internal Server Error
```json
{
  "error": "Failed to create credential",
  "details": "Insufficient XLM balance for contract deployment"
}
```

### Validation Errors

#### Invalid Contract ID
```json
{
  "error": "Invalid contract ID format"
}
```

#### Invalid Status
```json
{
  "error": "Valid status is required",
  "validStatuses": ["Active", "Revoked", "Suspended"]
}
```

#### Invalid Hash Format
```json
{
  "error": "Invalid hash format. Hash must be a 64-character hexadecimal string"
}
```

## Rate Limiting

The API implements rate limiting to prevent abuse:

- **General endpoints**: 100 requests per 15 minutes per IP
- **Credential creation**: 5 requests per minute per IP

Rate limit headers are included in responses:
```http
RateLimit-Limit: 100
RateLimit-Remaining: 95
RateLimit-Reset: 1642694400
```

## Request/Response Examples

### Complete Credential Creation Flow

1. **Create a credential:**
```bash
curl -X POST http://localhost:8000/credentials \
  -H "Content-Type: application/json" \
  -H "X-ACTA-Key: your_api_key_here" \
  -d '{
    "data": {
      "holder": "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "credentialType": "DriverLicense",
      "claims": {
        "licenseNumber": "DL123456789",
        "class": "Class A",
        "restrictions": "None",
        "expirationDate": "2029-01-15"
      }
    },
    "metadata": {
      "issuer": "Department of Motor Vehicles",
      "subject": "Jane Smith",
      "expirationDate": "2029-01-15T23:59:59Z"
    }
  }'
```

2. **Response:**
```json
{
  "success": true,
  "data": {
    "contractId": "CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ",
    "hash": "b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef1234567",
    "transactionHash": "8f9e0d1c2b3a4567890123456789012345678901abcdef234567890abcdef123",
    "ledgerSequence": 12345679,
    "createdAt": "2025-01-15T10:35:00.000Z"
  }
}
```

3. **Verify the credential:**
```bash
curl -X GET http://localhost:8000/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ
```

4. **Update credential status:**
```bash
curl -X PATCH http://localhost:8000/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ/status \
  -H "Content-Type: application/json" \
  -H "X-ACTA-Key: your_api_key_here" \
  -d '{"status": "Suspended"}'
```

## Testing the API

### Using Postman

1. Import the API collection (if available)
2. Set the base URL to your API endpoint
3. Configure environment variables for testing

### Using curl

All examples in this documentation use curl commands that you can run directly in your terminal.

### Using JavaScript/Node.js

```javascript
const axios = require('axios');

const API_BASE_URL = 'https://api.acta.build';

// Create a credential
async function createCredential(credentialData) {
  try {
    const response = await axios.post(`${API_BASE_URL}/credentials`, {
      data: credentialData
    });
    return response.data;
  } catch (error) {
    // Handle error appropriately in your application
    throw new Error(`Failed to create credential: ${error.response?.data?.message || error.message}`);
  }
}

// Get credential by contract ID
async function getCredential(contractId) {
  try {
    const response = await axios.get(`${API_BASE_URL}/credentials/${contractId}`);
    return response.data;
  } catch (error) {
    // Handle error appropriately in your application
    throw new Error(`Failed to get credential: ${error.response?.data?.message || error.message}`);
  }
}
```

---

*Next: Learn about [Services and Business Logic](./05-services.md) to understand how the API processes credentials.*