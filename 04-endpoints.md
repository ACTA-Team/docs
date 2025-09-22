# API Endpoints

## Overview

The ACTA API provides a comprehensive set of RESTful endpoints for managing verifiable credentials on the Stellar blockchain. All endpoints return JSON responses and follow standard HTTP status codes.

## Base URL

- **Development**: `http://localhost:3000`
- **Production**: `https://acta.up.railway.app`

## Response Format

All API responses follow a consistent format:

### Success Response
```json
{
  "success": true,
  "data": {
    // Response data
  }
}
```

### Error Response
```json
{
  "error": "Error message",
  "details": "Additional error details (development only)"
}
```

## Root Endpoints

### GET /

Returns basic API information and available endpoints.

**Request:**
```http
GET /
```

**Response:**
```json
{
  "message": "ACTA API - Stellar Credential Management",
  "version": "1.0.0",
  "endpoints": {
    "health": "/health",
    "credentials": "/v1/credentials"
  }
}
```

**Status Codes:**
- `200 OK`: Success

---

### GET /health

Health check endpoint for monitoring and load balancers.

**Request:**
```http
GET /health
```

**Response:**
```json
{
  "status": "OK",
  "timestamp": "2025-01-15T10:30:00.000Z",
  "service": "Stellar Credential API"
}
```

**Status Codes:**
- `200 OK`: Service is healthy
- `503 Service Unavailable`: Service is unhealthy

---

## Credentials Endpoints

All credential endpoints are prefixed with `/v1/credentials`.

### POST /v1/credentials

Creates a new verifiable credential on the Stellar blockchain.

**Request:**
```http
POST /v1/credentials
Content-Type: application/json

{
  "data": {
    "holder": "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "issuer": "University of Technology",
    "credentialType": "AcademicDegree",
    "subject": "Computer Science Degree",
    "issuanceDate": "2024-06-15",
    "claims": {
      "degree": "Bachelor of Science",
      "major": "Computer Science",
      "gpa": "3.8",
      "graduationYear": "2024"
    }
  },
  "metadata": {
    "issuer": "University of Technology",
    "subject": "John Doe",
    "expirationDate": "2029-06-15T23:59:59Z",
    "category": "education"
  }
}
```

**Request Body Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `data` | Object | Yes | The credential data to be stored |
| `metadata` | Object | No | Additional metadata about the credential |
| `metadata.issuer` | String | No | The issuer of the credential |
| `metadata.subject` | String | No | The subject of the credential |
| `metadata.expirationDate` | String | No | ISO 8601 expiration date |

**Response:**
```json
{
  "success": true,
  "data": {
    "contractId": "CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ",
    "hash": "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456",
    "transactionHash": "7f8e9d0c1b2a3456789012345678901234567890abcdef1234567890abcdef12",
    "ledgerSequence": 12345678,
    "createdAt": "2025-01-15T10:30:00.000Z"
  }
}
```

**Status Codes:**
- `201 Created`: Credential created successfully
- `400 Bad Request`: Invalid request data
- `500 Internal Server Error`: Server error

**Example cURL:**
```bash
curl -X POST https://acta.up.railway.app/v1/credentials \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "issuer": "GCKFBEIYTKP6RCZX6LROC7CWLZQYLKJ4KQPQKJLPQJLPQJLPQJLPQJLP",
    "subject": "GDQNYC2PDNPQN2VDMNCQJLPQJLPQJLPQJLPQJLPQJLPQJLPQJLPQJLP",
    "credentialData": {
      "name": "John Doe",
      "degree": "Bachelor of Science",
      "university": "Example University",
      "graduationDate": "2023-06-15"
    },
    "expirationDate": "2024-06-15T00:00:00Z"
  }'
```

---

### GET /v1/credentials/:contractId

Retrieves information about a specific credential by its contract ID.

**Request:**
```http
GET /v1/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ
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
curl -X GET https://acta.up.railway.app/v1/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ \
  -H "Authorization: Bearer YOUR_API_KEY"
```

---

### PATCH /v1/credentials/:contractId/status

Updates the status of an existing credential.

**Request:**
```http
PATCH /v1/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ/status
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
curl -X PATCH https://acta.up.railway.app/v1/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ/status \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"status": "revoked", "reason": "Credential no longer valid"}'
```

---

### GET /v1/credentials/hash/:hash

Retrieves credential information by its data hash.

**Request:**
```http
GET /v1/credentials/hash/a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456
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
curl -X GET https://acta.up.railway.app/v1/credentials/hash/a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456 \
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
  "path": "/v1/invalid-endpoint",
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
curl -X POST http://localhost:3000/v1/credentials \
  -H "Content-Type: application/json" \
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
curl -X GET http://localhost:3000/v1/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ
```

4. **Update credential status:**
```bash
curl -X PATCH http://localhost:3000/v1/credentials/CA2I6BAXNG7EHS4DF3JFXOQK3LSN6JULNVJ3GMHWTQAXI5WWP2VAEUIQ/status \
  -H "Content-Type: application/json" \
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

const API_BASE_URL = 'https://acta.up.railway.app/v1';

// Create a credential
async function createCredential(credentialData) {
  try {
    const response = await axios.post(`${API_BASE_URL}/v1/credentials`, {
      data: credentialData
    });
    return response.data;
  } catch (error) {
    console.error('Error creating credential:', error.response.data);
    throw error;
  }
}

// Get credential by contract ID
async function getCredential(contractId) {
  try {
    const response = await axios.get(`${API_BASE_URL}/v1/credentials/${contractId}`);
    return response.data;
  } catch (error) {
    console.error('Error getting credential:', error.response.data);
    throw error;
  }
}
```

---

*Next: Learn about [Services and Business Logic](./05-services.md) to understand how the API processes credentials.*