# Services and Business Logic

<div align="center">

![Services](https://img.shields.io/badge/Services-Business%20Logic-orange?style=for-the-badge&logo=gear&logoColor=white)

</div>

## Overview

The ACTA API is built with a modular service architecture that separates business logic from API endpoints. This section details the core services that power the credential management system.

---

## Service Architecture

### **Core Services**

| Service | Purpose | Dependencies |
|---------|---------|--------------|
| **CredentialService** | Manages credential lifecycle | StellarService, HashService |
| **StellarService** | Handles blockchain operations | Stellar SDK |
| **HashService** | Generates and validates hashes | crypto module |
| **ValidationService** | Validates input data | joi, custom validators |

---

## **CredentialService**

The main service responsible for credential management operations.

### **Methods**

#### `createCredential(data, metadata)`

Creates a new credential on the Stellar blockchain.

**Parameters:**
- `data` (Object): The credential data to store
- `metadata` (Object): Additional metadata

**Returns:**
- `Promise<Object>`: Created credential information

**Example:**
```javascript
const credentialService = new CredentialService();

const result = await credentialService.createCredential({
  type: "UniversityDegree",
  credentialSubject: {
    id: "did:example:123",
    degree: {
      type: "BachelorDegree",
      name: "Computer Science"
    }
  }
}, {
  issuer: "University of Technology",
  subject: "John Doe"
});
```

#### `getCredential(contractId)`

Retrieves credential information by contract ID.

**Parameters:**
- `contractId` (String): The Stellar contract ID

**Returns:**
- `Promise<Object>`: Credential information

#### `updateCredentialStatus(contractId, status)`

Updates the status of an existing credential.

**Parameters:**
- `contractId` (String): The Stellar contract ID
- `status` (String): New status ("Active", "Revoked", "Suspended")

**Returns:**
- `Promise<Object>`: Updated credential information

#### `getCredentialByHash(hash)`

Retrieves credential information by data hash.

**Parameters:**
- `hash` (String): SHA-256 hash of credential data

**Returns:**
- `Promise<Object>`: Credential information

---

## **StellarService**

Handles all Stellar blockchain interactions.

### **Configuration**

```javascript
const stellarService = new StellarService({
  network: 'testnet', // or 'mainnet'
  horizonUrl: 'https://horizon-testnet.stellar.org',
  secretKey: process.env.STELLAR_SECRET_KEY
});
```

### **Methods**

#### `deployContract(wasmHash, initArgs)`

Deploys a new smart contract to the Stellar network.

**Parameters:**
- `wasmHash` (String): Hash of the compiled WASM contract
- `initArgs` (Array): Initialization arguments

**Returns:**
- `Promise<String>`: Contract ID

#### `invokeContract(contractId, method, args)`

Invokes a method on an existing contract.

**Parameters:**
- `contractId` (String): The contract ID
- `method` (String): Method name to invoke
- `args` (Array): Method arguments

**Returns:**
- `Promise<Object>`: Transaction result

#### `getContractData(contractId, key)`

Retrieves data from a contract's storage.

**Parameters:**
- `contractId` (String): The contract ID
- `key` (String): Storage key

**Returns:**
- `Promise<any>`: Stored data

---

## **HashService**

Provides cryptographic hashing functionality.

### **Methods**

#### `generateHash(data)`

Generates a SHA-256 hash of the provided data.

**Parameters:**
- `data` (Object|String): Data to hash

**Returns:**
- `String`: 64-character hexadecimal hash

**Example:**
```javascript
const hashService = new HashService();
const hash = hashService.generateHash({
  type: "UniversityDegree",
  subject: "John Doe"
});
// Returns: "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456"
```

#### `validateHash(data, expectedHash)`

Validates that data matches the expected hash.

**Parameters:**
- `data` (Object|String): Data to validate
- `expectedHash` (String): Expected hash value

**Returns:**
- `Boolean`: True if hash matches

---

## **ValidationService**

Handles input validation and data sanitization.

### **Schemas**

#### Credential Data Schema

```javascript
const credentialSchema = {
  type: Joi.string().required(),
  credentialSubject: Joi.object().required(),
  issuer: Joi.string().required(),
  issuanceDate: Joi.string().isoDate().required()
};
```

#### Metadata Schema

```javascript
const metadataSchema = {
  issuer: Joi.string().optional(),
  subject: Joi.string().optional(),
  expirationDate: Joi.string().isoDate().optional(),
  category: Joi.string().optional()
};
```

### **Methods**

#### `validateCredentialData(data)`

Validates credential data against the schema.

**Parameters:**
- `data` (Object): Credential data to validate

**Returns:**
- `Object`: Validation result with errors if any

#### `validateContractId(contractId)`

Validates Stellar contract ID format.

**Parameters:**
- `contractId` (String): Contract ID to validate

**Returns:**
- `Boolean`: True if valid format

#### `validateHash(hash)`

Validates hash format (64-character hexadecimal).

**Parameters:**
- `hash` (String): Hash to validate

**Returns:**
- `Boolean`: True if valid format

---

## **Error Handling**

### **Service Errors**

All services use a consistent error handling approach:

```javascript
class ServiceError extends Error {
  constructor(message, code, statusCode = 500) {
    super(message);
    this.name = 'ServiceError';
    this.code = code;
    this.statusCode = statusCode;
  }
}
```

### **Common Error Codes**

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `VALIDATION_ERROR` | Input validation failed | 400 |
| `STELLAR_ERROR` | Stellar network error | 500 |
| `CONTRACT_NOT_FOUND` | Contract doesn't exist | 404 |
| `INSUFFICIENT_BALANCE` | Not enough XLM for operation | 400 |
| `NETWORK_ERROR` | Network connectivity issue | 503 |

---

## **Service Integration**

### **Dependency Injection**

Services are injected into controllers using a simple DI container:

```javascript
class ServiceContainer {
  constructor() {
    this.services = new Map();
  }

  register(name, service) {
    this.services.set(name, service);
  }

  get(name) {
    return this.services.get(name);
  }
}

// Usage
const container = new ServiceContainer();
container.register('credentialService', new CredentialService());
container.register('stellarService', new StellarService());
```

### **Service Lifecycle**

1. **Initialization**: Services are created with configuration
2. **Registration**: Services are registered in the DI container
3. **Injection**: Controllers receive service instances
4. **Execution**: Services handle business logic
5. **Cleanup**: Resources are properly disposed

---

## **Testing Services**

### **Unit Testing**

Each service includes comprehensive unit tests:

```javascript
describe('CredentialService', () => {
  let credentialService;
  let mockStellarService;

  beforeEach(() => {
    mockStellarService = {
      deployContract: jest.fn(),
      invokeContract: jest.fn()
    };
    credentialService = new CredentialService(mockStellarService);
  });

  test('should create credential successfully', async () => {
    mockStellarService.deployContract.mockResolvedValue('CONTRACT_ID');
    
    const result = await credentialService.createCredential({
      type: 'TestCredential'
    });

    expect(result.contractId).toBe('CONTRACT_ID');
  });
});
```

### **Integration Testing**

Integration tests verify service interactions:

```javascript
describe('Service Integration', () => {
  test('should create and retrieve credential', async () => {
    const credentialService = new CredentialService();
    
    // Create credential
    const created = await credentialService.createCredential(testData);
    
    // Retrieve credential
    const retrieved = await credentialService.getCredential(created.contractId);
    
    expect(retrieved.hash).toBe(created.hash);
  });
});
```

---

## **Performance Considerations**

### **Caching Strategy**

- **Contract Data**: Cache frequently accessed contract data
- **Hash Calculations**: Cache computed hashes for large datasets
- **Validation Results**: Cache validation results for repeated requests

### **Optimization Techniques**

1. **Batch Operations**: Group multiple blockchain operations
2. **Connection Pooling**: Reuse Stellar network connections
3. **Async Processing**: Use queues for non-critical operations
4. **Data Compression**: Compress large credential data

---

## **Monitoring and Logging**

### **Service Metrics**

Key metrics tracked for each service:

- **Response Time**: Average and 95th percentile
- **Error Rate**: Percentage of failed operations
- **Throughput**: Operations per second
- **Resource Usage**: Memory and CPU consumption

### **Logging Strategy**

```javascript
const logger = require('./utils/logger');

class CredentialService {
  async createCredential(data) {
    logger.info('Creating credential', { 
      type: data.type,
      timestamp: new Date().toISOString()
    });

    try {
      const result = await this.processCredential(data);
      logger.info('Credential created successfully', { 
        contractId: result.contractId 
      });
      return result;
    } catch (error) {
      logger.error('Failed to create credential', { 
        error: error.message,
        stack: error.stack 
      });
      throw error;
    }
  }
}
```

---

*Next: Learn about [Configuration](./06-configuration.md) to understand how to configure the API for different environments.*