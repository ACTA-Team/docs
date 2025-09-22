# Troubleshooting and FAQ

## Overview

This section provides solutions to common issues, error messages, and frequently asked questions when working with the ACTA API. Use this guide to quickly resolve problems and understand best practices.

## Common Issues and Solutions

### 1. Stellar Network Connection Issues

#### Problem: "Failed to connect to Stellar Horizon"

**Symptoms:**
- API returns 500 errors
- Health check fails
- Stellar operations timeout

**Solutions:**

1. **Check Network Configuration:**
```bash
# Verify Horizon URL is accessible
curl -I https://horizon.stellar.org

# Check environment variables
echo $STELLAR_HORIZON_URL
echo $STELLAR_NETWORK_PASSPHRASE
```

2. **Validate Network Passphrase:**
```javascript
// Testnet
STELLAR_NETWORK_PASSPHRASE=Test SDF Network ; September 2015

// Mainnet
STELLAR_NETWORK_PASSPHRASE=Public Global Stellar Network ; September 2015
```

3. **Test Connection Programmatically:**
```javascript
const { Server } = require('stellar-sdk');

async function testConnection() {
  try {
    const server = new Server(process.env.STELLAR_HORIZON_URL);
    const ledger = await server.ledgers().limit(1).call();
    console.log('✅ Connection successful:', ledger.records[0].sequence);
  } catch (error) {
    console.error('❌ Connection failed:', error.message);
  }
}
```

#### Problem: "Account not found" or "Invalid account"

**Symptoms:**
- 404 errors when loading account
- Operations fail with account errors

**Solutions:**

1. **Verify Account Exists:**
```bash
# Check account on Stellar Expert
# Testnet: https://stellar.expert/explorer/testnet/account/YOUR_PUBLIC_KEY
# Mainnet: https://stellar.expert/explorer/public/account/YOUR_PUBLIC_KEY
```

2. **Fund Account (Testnet):**
```javascript
// For testnet only
const response = await fetch(`https://friendbot.stellar.org?addr=${publicKey}`);
console.log('Account funded:', response.ok);
```

3. **Check Account Balance:**
```javascript
const account = await server.loadAccount(publicKey);
console.log('Balances:', account.balances);
```

### 2. Smart Contract Issues

#### Problem: "Contract not found" or "Invalid contract ID"

**Symptoms:**
- Contract operations fail
- 404 errors when calling contract methods

**Solutions:**

1. **Verify Contract Deployment:**
```javascript
const { SorobanRpc } = require('stellar-sdk');

async function checkContract(contractId) {
  try {
    const server = new SorobanRpc.Server(process.env.STELLAR_SOROBAN_URL);
    const contract = await server.getContractData(contractId);
    console.log('✅ Contract found:', contract);
  } catch (error) {
    console.error('❌ Contract not found:', error.message);
  }
}
```

2. **Validate Contract ID Format:**
```javascript
// Contract ID should be 56 characters long and start with 'C'
const isValidContractId = (id) => {
  return /^C[A-Z0-9]{55}$/.test(id);
};
```

3. **Check Contract Methods:**
```javascript
// List available contract methods
const contractSpec = await server.getContractSpec(contractId);
console.log('Available methods:', contractSpec.methods);
```

#### Problem: "Transaction failed" or "Soroban transaction error"

**Symptoms:**
- Contract calls return errors
- Transactions are rejected

**Solutions:**

1. **Check Transaction Fees:**
```javascript
// Increase fee for complex operations
const transaction = new TransactionBuilder(account, {
  fee: '1000000', // 0.1 XLM
  networkPassphrase: Networks.TESTNET
});
```

2. **Validate Function Parameters:**
```javascript
// Ensure parameters match contract expectations
const params = [
  nativeToScVal('credential_id', { type: 'string' }),
  nativeToScVal(credentialData, { type: 'map' })
];
```

3. **Check Account Authorization:**
```javascript
// Verify account has necessary permissions
const account = await server.loadAccount(sourceKeypair.publicKey());
console.log('Account signers:', account.signers);
```

### 3. API Authentication and Authorization

#### Problem: "Unauthorized" or "Forbidden" errors

**Symptoms:**
- 401/403 HTTP status codes
- Access denied messages

**Solutions:**

1. **Verify API Key (if implemented):**
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     http://localhost:3000/api/credentials
```

2. **Check CORS Configuration:**
```javascript
// Ensure your domain is in allowed origins
const corsOptions = {
  origin: ['https://yourdomain.com', 'http://localhost:3000'],
  credentials: true
};
```

3. **Validate Request Headers:**
```javascript
// Required headers for API requests
const headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
};
```

### 4. Rate Limiting Issues

#### Problem: "Too Many Requests" (429 errors)

**Symptoms:**
- API returns 429 status code
- Requests are being throttled

**Solutions:**

1. **Implement Exponential Backoff:**
```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

2. **Check Rate Limit Headers:**
```javascript
// Monitor rate limit status
const response = await fetch('/api/credentials');
console.log('Rate limit remaining:', response.headers.get('X-RateLimit-Remaining'));
console.log('Rate limit reset:', response.headers.get('X-RateLimit-Reset'));
```

3. **Optimize Request Patterns:**
```javascript
// Batch operations instead of individual requests
const credentials = await Promise.all([
  createCredential(data1),
  createCredential(data2),
  createCredential(data3)
]);
```

### 5. Data Validation Errors

#### Problem: "Invalid input data" or validation errors

**Symptoms:**
- 400 Bad Request errors
- Validation error messages

**Solutions:**

1. **Validate Credential Data Structure:**
```javascript
const credentialSchema = {
  recipient: 'string', // Required
  issuer: 'string',    // Required
  credential_type: 'string', // Required
  metadata: 'object'   // Optional
};

function validateCredential(data) {
  const required = ['recipient', 'issuer', 'credential_type'];
  const missing = required.filter(field => !data[field]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required fields: ${missing.join(', ')}`);
  }
  
  return true;
}
```

2. **Check Data Types:**
```javascript
// Ensure proper data types
const credentialData = {
  recipient: String(recipient),
  issuer: String(issuer),
  credential_type: String(credentialType),
  metadata: typeof metadata === 'object' ? metadata : {}
};
```

3. **Validate Stellar Addresses:**
```javascript
const { StrKey } = require('stellar-sdk');

function isValidStellarAddress(address) {
  try {
    return StrKey.isValidEd25519PublicKey(address);
  } catch {
    return false;
  }
}
```

### 6. Performance Issues

#### Problem: Slow API responses or timeouts

**Symptoms:**
- Long response times
- Request timeouts
- High server load

**Solutions:**

1. **Implement Caching:**
```javascript
// Cache frequently accessed data
const cache = new Map();

async function getCachedCredential(hash) {
  if (cache.has(hash)) {
    return cache.get(hash);
  }
  
  const credential = await fetchCredential(hash);
  cache.set(hash, credential);
  
  // Auto-expire after 5 minutes
  setTimeout(() => cache.delete(hash), 5 * 60 * 1000);
  
  return credential;
}
```

2. **Optimize Database Queries:**
```javascript
// Use indexes for frequently queried fields
// Add pagination for large result sets
const credentials = await getCredentials({
  limit: 50,
  offset: page * 50,
  orderBy: 'created_at',
  order: 'DESC'
});
```

3. **Monitor Resource Usage:**
```javascript
// Check memory usage
const memUsage = process.memoryUsage();
console.log('Memory usage:', {
  rss: `${Math.round(memUsage.rss / 1024 / 1024)}MB`,
  heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)}MB`
});
```

## Error Code Reference

### HTTP Status Codes

| Code | Meaning | Common Causes | Solutions |
|------|---------|---------------|-----------|
| 400 | Bad Request | Invalid input data, malformed JSON | Validate request body and parameters |
| 401 | Unauthorized | Missing or invalid authentication | Check API keys or authentication headers |
| 403 | Forbidden | Insufficient permissions | Verify account permissions and access rights |
| 404 | Not Found | Resource doesn't exist | Check resource IDs and endpoints |
| 409 | Conflict | Resource already exists | Use different identifier or update existing |
| 429 | Too Many Requests | Rate limit exceeded | Implement backoff strategy |
| 500 | Internal Server Error | Server-side error | Check logs and server health |
| 503 | Service Unavailable | Server overloaded or maintenance | Retry later or check service status |

### Stellar-Specific Errors

| Error | Description | Solution |
|-------|-------------|----------|
| `tx_failed` | Transaction failed | Check transaction parameters and account balance |
| `tx_bad_seq` | Bad sequence number | Reload account and use current sequence |
| `tx_insufficient_balance` | Insufficient XLM balance | Fund account with more XLM |
| `op_no_destination` | Destination account doesn't exist | Create or fund destination account |
| `op_not_supported` | Operation not supported | Check operation type and parameters |

### Contract-Specific Errors

| Error | Description | Solution |
|-------|-------------|----------|
| `ContractNotFound` | Contract doesn't exist | Verify contract ID and deployment |
| `FunctionNotFound` | Contract function doesn't exist | Check function name and contract spec |
| `InvalidArgument` | Invalid function argument | Validate argument types and values |
| `InsufficientFunds` | Not enough funds for operation | Ensure account has sufficient balance |

## Frequently Asked Questions (FAQ)

### General Questions

**Q: What is the ACTA API?**
A: The ACTA API is a credential management system built on the Stellar blockchain that allows for the creation, verification, and management of digital credentials using smart contracts.

**Q: Which Stellar network should I use?**
A: Use Testnet for development and testing, and Mainnet for production. Never use Mainnet for testing as it involves real XLM costs.

**Q: How much does it cost to use the API?**
A: The API itself is free to use, but Stellar network operations require XLM for transaction fees. Typical operations cost 0.00001 XLM (100 stroops) plus additional fees for smart contract operations.

### Technical Questions

**Q: How do I generate a Stellar keypair?**
A: Use the Stellar SDK:
```javascript
const { Keypair } = require('stellar-sdk');
const keypair = Keypair.random();
console.log('Public Key:', keypair.publicKey());
console.log('Secret Key:', keypair.secret());
```

**Q: Can I use the API without a Stellar account?**
A: No, you need a funded Stellar account to perform operations. For testnet, you can use Friendbot to fund your account.

**Q: How do I backup my Stellar keys?**
A: Store your secret key securely offline. Never share it or commit it to version control. Consider using hardware wallets for production keys.

**Q: What happens if I lose my secret key?**
A: If you lose your secret key, you cannot recover it or access your account. Always keep secure backups of your keys.

**Q: How do I update a credential?**
A: Use the PATCH endpoint to update credential status:
```bash
curl -X PATCH http://localhost:3000/api/credentials/CONTRACT_ID/status \
  -H "Content-Type: application/json" \
  -d '{"status": "Revoked"}'
```

**Q: Can I delete a credential?**
A: Credentials on the blockchain are immutable, but you can revoke them by updating their status to "Revoked".

**Q: How do I verify a credential?**
A: Use the verification endpoint with the credential hash:
```bash
curl http://localhost:3000/api/credentials/hash/CREDENTIAL_HASH
```

### Development Questions

**Q: How do I set up a development environment?**
A: Follow the setup guide in the documentation. Use Docker for the easiest setup:
```bash
git clone <repository>
cd acta-api
cp .env.example .env
docker-compose up -d
```

**Q: How do I run tests?**
A: Use npm to run the test suite:
```bash
npm test                # Run all tests
npm run test:unit      # Run unit tests only
npm run test:integration # Run integration tests only
```

**Q: How do I enable debug logging?**
A: Set the LOG_LEVEL environment variable:
```bash
LOG_LEVEL=debug npm start
```

**Q: Can I use the API with other programming languages?**
A: Yes, the API is language-agnostic. Any language that can make HTTP requests can use the API. We provide examples for JavaScript, Python, and cURL.

### Production Questions

**Q: How do I deploy to production?**
A: Follow the deployment guide. Key steps include:
1. Set up production environment variables
2. Use HTTPS with valid SSL certificates
3. Configure proper CORS settings
4. Set up monitoring and logging
5. Implement backup strategies

**Q: How do I monitor the API in production?**
A: Use the built-in health check endpoint and implement monitoring:
```bash
# Health check
curl http://your-api.com/health

# Metrics endpoint (if enabled)
curl http://your-api.com/metrics
```

**Q: How do I handle high traffic?**
A: Implement horizontal scaling:
- Use load balancers
- Deploy multiple API instances
- Implement caching
- Use CDN for static assets
- Monitor and optimize database queries

**Q: What are the security best practices?**
A: Follow these security guidelines:
- Use HTTPS in production
- Implement rate limiting
- Validate all inputs
- Use secure headers
- Keep dependencies updated
- Monitor for security vulnerabilities
- Use environment variables for secrets

### Troubleshooting Workflow

When encountering issues, follow this systematic approach:

1. **Check the Logs:**
   ```bash
   # Application logs
   tail -f logs/combined.log
   
   # Error logs
   tail -f logs/error.log
   
   # Docker logs
   docker logs acta-api
   ```

2. **Verify Configuration:**
   ```bash
   # Check environment variables
   env | grep STELLAR
   
   # Test configuration
   npm run config:test
   ```

3. **Test Connectivity:**
   ```bash
   # Test Stellar connection
   curl -I https://horizon.stellar.org
   
   # Test API health
   curl http://localhost:3000/health
   ```

4. **Check Resources:**
   ```bash
   # Memory usage
   free -h
   
   # Disk space
   df -h
   
   # Process status
   ps aux | grep node
   ```

5. **Review Recent Changes:**
   - Check recent deployments
   - Review configuration changes
   - Verify dependency updates

If issues persist, check the GitHub issues page or contact support with:
- Error messages and logs
- Steps to reproduce
- Environment details
- API version

---

*This concludes the ACTA API documentation. For additional support, please refer to the [GitHub repository](https://github.com/your-org/acta-api) or contact the development team.*