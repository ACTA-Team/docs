# Services and Business Logic

## Overview

The ACTA API is built with a service-oriented architecture that separates business logic from API routing. The main service component is the `StellarService`, which handles all interactions with the Stellar blockchain and smart contract operations.

## StellarService

The `StellarService` class is the core component responsible for managing credentials on the Stellar blockchain. It provides methods for creating, retrieving, and updating verifiable credentials using Stellar's Soroban smart contracts.

### Class Structure

```typescript
class StellarService {
  private server: Server;
  private sorobanServer: SorobanRpc.Server;
  private sourceKeypair: Keypair;
  private networkPassphrase: string;
  private contractId: string;

  constructor(config: StellarConfig)
  // Methods for credential management
}
```

### Configuration

The service is initialized with a configuration object containing:

| Property | Type | Description |
|----------|------|-------------|
| `secretKey` | string | Stellar secret key for transaction signing |
| `horizonUrl` | string | Horizon server URL |
| `sorobanUrl` | string | Soroban RPC server URL |
| `networkPassphrase` | string | Network identifier |

### Core Methods

#### generateHash(data: any): string

Generates a SHA-256 hash of the credential data for blockchain storage.

**Parameters:**
- `data`: The credential data object to hash

**Returns:**
- A 64-character hexadecimal hash string

**Example:**
```typescript
const hash = stellarService.generateHash({
  holder: "GXXXXXXX...",
  credentialType: "AcademicDegree",
  claims: { degree: "Bachelor of Science" }
});
// Returns: "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456"
```

#### deployContract(): Promise<string>

Deploys a new smart contract instance on the Stellar network.

**Returns:**
- Promise resolving to the contract ID

**Process:**
1. Loads the WASM contract bytecode
2. Creates a deployment transaction
3. Submits to the network
4. Returns the generated contract ID

**Note:** Currently uses a fixed contract ID for all operations.

#### createCredential(data: any, metadata?: any): Promise<CredentialResponse>

Creates a new verifiable credential on the blockchain.

**Parameters:**
- `data`: The credential data to store
- `metadata`: Optional metadata about the credential

**Returns:**
- Promise resolving to a `CredentialResponse` object

**Process:**
1. Validates the source account balance (minimum 10 XLM required)
2. Generates a hash of the credential data
3. Creates a smart contract transaction
4. Submits the transaction to the network
5. Returns transaction details and credential information

**Example:**
```typescript
const credential = await stellarService.createCredential({
  holder: "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  credentialType: "DriverLicense",
  claims: {
    licenseNumber: "DL123456789",
    class: "Class A"
  }
}, {
  issuer: "DMV",
  expirationDate: "2029-01-15T23:59:59Z"
});
```

#### getCredential(contractId: string): Promise<any>

Retrieves credential information from the blockchain.

**Parameters:**
- `contractId`: The Stellar contract ID

**Returns:**
- Promise resolving to credential data

**Process:**
1. Validates the contract ID format
2. Queries the smart contract state
3. Returns the stored credential information

#### updateCredentialStatus(contractId: string, status: CredentialStatus): Promise<void>

Updates the status of an existing credential.

**Parameters:**
- `contractId`: The Stellar contract ID
- `status`: New status (Active, Revoked, or Suspended)

**Returns:**
- Promise that resolves when the update is complete

**Process:**
1. Validates the contract ID and status
2. Creates an update transaction
3. Submits to the network
4. Confirms the transaction

#### getCredentialByHash(hash: string): Promise<any>

Retrieves credential information using its data hash.

**Parameters:**
- `hash`: SHA-256 hash of the credential data

**Returns:**
- Promise resolving to credential data

**Process:**
1. Validates the hash format (64 hex characters)
2. Queries the blockchain for matching credentials
3. Returns the credential information

## Error Handling

The service implements comprehensive error handling for various scenarios:

### Balance Validation

```typescript
if (account.balances[0].balance < '10') {
  throw new Error('Insufficient XLM balance. Minimum 10 XLM required for contract operations.');
}
```

### Contract ID Validation

```typescript
if (!contractId || contractId.length !== 56 || !contractId.startsWith('C')) {
  throw new Error('Invalid contract ID format');
}
```

### Hash Validation

```typescript
if (!hash || hash.length !== 64 || !/^[a-fA-F0-9]+$/.test(hash)) {
  throw new Error('Invalid hash format. Hash must be a 64-character hexadecimal string');
}
```

### Network Errors

The service handles various network-related errors:
- Connection timeouts
- Invalid responses
- Transaction failures
- Insufficient fees

## Smart Contract Integration

### Contract Deployment

The service manages smart contract deployment with the following features:

- **WASM Loading**: Loads contract bytecode from the filesystem
- **Transaction Building**: Creates deployment transactions with proper fees
- **Network Submission**: Submits transactions to the Stellar network
- **Result Processing**: Extracts contract IDs from transaction results

### Contract Interaction

#### Method Invocation

The service interacts with deployed contracts through method calls:

```typescript
// Example contract method call structure
const transaction = new TransactionBuilder(account, {
  fee: BASE_FEE,
  networkPassphrase: this.networkPassphrase,
})
.addOperation(
  Operation.invokeContract({
    contract: this.contractId,
    method: 'store_credential',
    args: [
      nativeToScVal(hash, { type: 'string' }),
      nativeToScVal(data, { type: 'string' })
    ]
  })
)
.setTimeout(30)
.build();
```

#### State Queries

Reading contract state to retrieve stored credentials:

```typescript
const result = await this.sorobanServer.getContractData(
  this.contractId,
  nativeToScVal(hash, { type: 'string' })
);
```

## Data Types and Interfaces

### CredentialRequest

```typescript
interface CredentialRequest {
  data: any;
  metadata?: {
    issuer?: string;
    subject?: string;
    expirationDate?: string;
    category?: string;
  };
}
```

### CredentialResponse

```typescript
interface CredentialResponse {
  contractId: string;
  hash: string;
  transactionHash: string;
  ledgerSequence: number;
  createdAt: string;
}
```

### CredentialStatus

```typescript
enum CredentialStatus {
  Active = 'Active',
  Revoked = 'Revoked',
  Suspended = 'Suspended'
}
```

### StellarConfig

```typescript
interface StellarConfig {
  secretKey: string;
  horizonUrl: string;
  sorobanUrl: string;
  networkPassphrase: string;
}
```

## Service Initialization

The service is initialized in the credentials router with environment variable validation:

```typescript
// Environment validation
const requiredEnvVars = ['STELLAR_SECRET_KEY', 'STELLAR_HORIZON_URL'];
const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  throw new Error(`Missing required environment variables: ${missingVars.join(', ')}`);
}

// Service initialization
const stellarService = new StellarService({
  secretKey: process.env.STELLAR_SECRET_KEY!,
  horizonUrl: process.env.STELLAR_HORIZON_URL!,
  sorobanUrl: process.env.STELLAR_SOROBAN_URL || 'https://soroban-testnet.stellar.org',
  networkPassphrase: process.env.STELLAR_NETWORK_PASSPHRASE || Networks.TESTNET
});
```

## Performance Considerations

### Transaction Fees

The service manages transaction fees efficiently:
- Uses base fees for standard operations
- Implements fee estimation for complex transactions
- Handles fee bumping for failed transactions

### Connection Pooling

- Maintains persistent connections to Horizon and Soroban servers
- Implements connection retry logic
- Handles server failover scenarios

### Caching

While not implemented in the current version, the service architecture supports:
- Contract state caching
- Transaction result caching
- Account balance caching

## Security Features

### Key Management

- Secure handling of Stellar secret keys
- Environment-based key configuration
- No key exposure in logs or responses

### Input Validation

- Comprehensive validation of all input parameters
- Sanitization of user-provided data
- Protection against injection attacks

### Transaction Security

- Proper transaction signing
- Sequence number management
- Timeout handling for pending transactions

## Testing the Service

### Unit Testing

```typescript
describe('StellarService', () => {
  let service: StellarService;

  beforeEach(() => {
    service = new StellarService({
      secretKey: 'SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
      horizonUrl: 'https://horizon-testnet.stellar.org',
      sorobanUrl: 'https://soroban-testnet.stellar.org',
      networkPassphrase: Networks.TESTNET
    });
  });

  it('should generate consistent hashes', () => {
    const data = { test: 'data' };
    const hash1 = service.generateHash(data);
    const hash2 = service.generateHash(data);
    expect(hash1).toBe(hash2);
  });
});
```

### Integration Testing

```typescript
describe('StellarService Integration', () => {
  it('should create and retrieve credentials', async () => {
    const credentialData = {
      holder: 'GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
      credentialType: 'TestCredential'
    };

    const result = await service.createCredential(credentialData);
    expect(result.contractId).toBeDefined();
    expect(result.hash).toBeDefined();

    const retrieved = await service.getCredential(result.contractId);
    expect(retrieved).toBeDefined();
  });
});
```

## Future Enhancements

### Planned Features

1. **Batch Operations**: Support for creating multiple credentials in a single transaction
2. **Event Streaming**: Real-time updates for credential status changes
3. **Advanced Querying**: Support for complex credential searches
4. **Backup and Recovery**: Automated backup of critical credential data

### Performance Improvements

1. **Connection Pooling**: Implement connection pooling for better performance
2. **Caching Layer**: Add Redis-based caching for frequently accessed data
3. **Async Processing**: Background processing for non-critical operations

---

*Next: Learn about [Configuration and Environment Variables](./06-configuration.md) to properly set up your API environment.*