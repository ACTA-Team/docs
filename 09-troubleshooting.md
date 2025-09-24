# Troubleshooting and FAQ

## Overview

This section provides solutions to common issues, debugging techniques, and frequently asked questions about the ACTA API. Use this guide to quickly resolve problems and optimize your implementation.

---

## **Common Setup Issues**

### **Installation Problems**

#### **Node.js Version Compatibility**

**Problem**: API fails to start with Node.js version errors.

```bash
Error: The engine "node" is incompatible with this module
```

**Solution**:
```bash
# Check your Node.js version
node --version

# Install the correct version (18.x or higher)
nvm install 18
nvm use 18

# Or using Node Version Manager on Windows
nvm install 18.17.0
nvm use 18.17.0
```

#### **Dependency Installation Failures**

**Problem**: npm install fails with permission or network errors.

```bash
npm ERR! code EACCES
npm ERR! syscall access
```

**Solutions**:
```bash
# Clear npm cache
npm cache clean --force

# Use npm ci for clean install
npm ci

# Fix permissions (Linux/macOS)
sudo chown -R $(whoami) ~/.npm

# Use different registry if network issues
npm install --registry https://registry.npmjs.org/
```

#### **Missing Environment Variables**

**Problem**: API crashes on startup due to missing configuration.

```bash
Error: STELLAR_SECRET_KEY is required
```

**Solution**:
```bash
# Create .env file with required variables
cp .env.example .env

# Edit .env file with your values
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_SOROBAN_URL=https://soroban-testnet.stellar.org
```

---

## **Stellar Network Issues**

### **Connection Problems**

#### **Horizon Server Unreachable**

**Problem**: Cannot connect to Stellar Horizon server.

```javascript
Error: Network Error: Request failed with status code 503
```

**Diagnosis**:
```bash
# Test Horizon connectivity
curl -I https://horizon-testnet.stellar.org/

# Check if using correct network
echo $STELLAR_HORIZON_URL
```

**Solutions**:
```javascript
// Use backup Horizon servers
const horizonUrls = [
  'https://horizon-testnet.stellar.org',
  'https://horizon.stellar.org' // fallback
];

// Implement retry logic
async function connectWithRetry(urls, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    for (const url of urls) {
      try {
        const server = new Server(url);
        await server.ledgers().limit(1).call();
        return server;
      } catch (error) {
        console.warn(`Failed to connect to ${url}:`, error.message);
      }
    }
    await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
  }
  throw new Error('All Horizon servers unreachable');
}
```

#### **Soroban RPC Issues**

**Problem**: Smart contract calls fail with RPC errors.

```javascript
Error: soroban-rpc: transaction submission failed
```

**Diagnosis**:
```bash
# Test Soroban RPC
curl -X POST https://soroban-testnet.stellar.org \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getHealth"}'
```

**Solutions**:
```javascript
// Configure Soroban with proper timeout
const sorobanServer = new SorobanRpc.Server(
  process.env.STELLAR_SOROBAN_URL,
  {
    allowHttp: false,
    timeout: 30000 // 30 seconds
  }
);

// Add retry mechanism for contract calls
async function callContractWithRetry(operation, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.warn(`Contract call attempt ${i + 1} failed:`, error.message);
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
}
```

### **Account and Key Issues**

#### **Invalid Secret Key Format**

**Problem**: Secret key validation fails.

```javascript
Error: Invalid secret key format
```

**Solution**:
```javascript
// Validate secret key format
function validateSecretKey(secretKey) {
  try {
    const keypair = Keypair.fromSecret(secretKey);
    console.log('Valid secret key for account:', keypair.publicKey());
    return true;
  } catch (error) {
    console.error('Invalid secret key:', error.message);
    return false;
  }
}

// Check key format
if (!validateSecretKey(process.env.STELLAR_SECRET_KEY)) {
  throw new Error('Please provide a valid Stellar secret key');
}
```

#### **Insufficient Account Balance**

**Problem**: Transactions fail due to insufficient XLM balance.

```javascript
Error: tx_insufficient_balance
```

**Diagnosis**:
```javascript
// Check account balance
async function checkAccountBalance(publicKey) {
  try {
    const server = new Server(process.env.STELLAR_HORIZON_URL);
    const account = await server.loadAccount(publicKey);
    
    console.log('Account balances:');
    account.balances.forEach(balance => {
      console.log(`${balance.asset_type}: ${balance.balance}`);
    });
    
    return account.balances;
  } catch (error) {
    console.error('Failed to load account:', error.message);
    throw error;
  }
}
```

**Solutions**:
```javascript
// Fund testnet account
async function fundTestnetAccount(publicKey) {
  try {
    const response = await fetch(
      `https://friendbot.stellar.org?addr=${publicKey}`
    );
    
    if (response.ok) {
      console.log('Account funded successfully');
    } else {
      throw new Error('Failed to fund account');
    }
  } catch (error) {
    console.error('Funding failed:', error.message);
    throw error;
  }
}

// For mainnet, ensure sufficient XLM balance
const MIN_BALANCE = 10; // XLM
async function ensureSufficientBalance(account) {
  const xlmBalance = account.balances.find(b => b.asset_type === 'native');
  const balance = parseFloat(xlmBalance.balance);
  
  if (balance < MIN_BALANCE) {
    throw new Error(`Insufficient balance: ${balance} XLM (minimum: ${MIN_BALANCE} XLM)`);
  }
}
```

---

## **Smart Contract Issues**

### **Contract Deployment Problems**

#### **Contract Compilation Errors**

**Problem**: Smart contract fails to compile.

```bash
Error: compilation failed
```

**Solutions**:
```bash
# Check Rust toolchain
rustc --version
cargo --version

# Update Rust and Soroban CLI
rustup update
cargo install --locked soroban-cli

# Clean and rebuild
cargo clean
soroban contract build
```

#### **Contract Deployment Failures**

**Problem**: Contract deployment transaction fails.

```javascript
Error: Contract deployment failed with status: failed
```

**Diagnosis**:
```bash
# Check contract size
ls -la target/wasm32-unknown-unknown/release/*.wasm

# Verify contract is optimized
soroban contract optimize --wasm target/wasm32-unknown-unknown/release/contract.wasm
```

**Solutions**:
```javascript
// Increase transaction timeout for deployment
const transaction = new TransactionBuilder(account, {
  fee: BASE_FEE,
  networkPassphrase: Networks.TESTNET,
  timebounds: {
    minTime: 0,
    maxTime: Math.floor(Date.now() / 1000) + 300 // 5 minutes
  }
});

// Add proper error handling
try {
  const result = await server.submitTransaction(transaction);
  console.log('Contract deployed:', result.hash);
} catch (error) {
  if (error.response?.data?.extras?.result_codes) {
    console.error('Transaction failed:', error.response.data.extras.result_codes);
  }
  throw error;
}
```

### **Contract Invocation Issues**

#### **Function Call Failures**

**Problem**: Contract function calls return errors.

```javascript
Error: Contract function 'create_credential' failed
```

**Debugging**:
```javascript
// Add detailed logging for contract calls
async function debugContractCall(contractId, functionName, args) {
  console.log('Contract Call Debug:');
  console.log('- Contract ID:', contractId);
  console.log('- Function:', functionName);
  console.log('- Arguments:', args);
  
  try {
    const result = await contract.call(functionName, ...args);
    console.log('- Result:', result);
    return result;
  } catch (error) {
    console.error('- Error:', error.message);
    if (error.response) {
      console.error('- Response:', error.response.data);
    }
    throw error;
  }
}
```

**Solutions**:
```javascript
// Implement proper argument encoding
function encodeContractArgs(args) {
  return args.map(arg => {
    if (typeof arg === 'string') {
      return nativeToScVal(arg, { type: 'string' });
    } else if (typeof arg === 'number') {
      return nativeToScVal(arg, { type: 'u64' });
    } else if (Buffer.isBuffer(arg)) {
      return nativeToScVal(arg, { type: 'bytes' });
    }
    return arg;
  });
}

// Add retry logic for contract calls
async function callContractWithRetry(contract, method, args, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await contract.call(method, ...encodeContractArgs(args));
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.warn(`Contract call retry ${i + 1}:`, error.message);
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
}
```

---

## **API Endpoint Issues**

### **Authentication Problems**

#### **Invalid API Key**

**Problem**: Requests fail with authentication errors.

```json
{
  "error": "Invalid API key",
  "code": 401
}
```

**Solutions**:
```javascript
// Verify API key format
function validateApiKey(apiKey) {
  if (!apiKey || typeof apiKey !== 'string') {
    throw new Error('API key must be a non-empty string');
  }
  
  if (apiKey.length < 32) {
    throw new Error('API key appears to be too short');
  }
  
  return true;
}

// Check API key in request headers
app.use('/api', (req, res, next) => {
  const apiKey = req.headers['x-api-key'] || req.headers['authorization']?.replace('Bearer ', '');
  
  if (!validateApiKey(apiKey)) {
    return res.status(401).json({ error: 'Invalid API key format' });
  }
  
  next();
});
```

#### **Token Expiration**

**Problem**: JWT tokens expire during long operations.

```json
{
  "error": "Token expired",
  "code": 401
}
```

**Solutions**:
```javascript
// Implement token refresh
async function refreshToken(oldToken) {
  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${oldToken}`,
        'Content-Type': 'application/json'
      }
    });
    
    if (response.ok) {
      const { token } = await response.json();
      return token;
    }
    
    throw new Error('Token refresh failed');
  } catch (error) {
    console.error('Token refresh error:', error.message);
    throw error;
  }
}

// Auto-retry with token refresh
async function apiCallWithRetry(url, options, maxRetries = 1) {
  for (let i = 0; i <= maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      
      if (response.status === 401 && i < maxRetries) {
        // Try to refresh token
        const newToken = await refreshToken(options.headers.Authorization.replace('Bearer ', ''));
        options.headers.Authorization = `Bearer ${newToken}`;
        continue;
      }
      
      return response;
    } catch (error) {
      if (i === maxRetries) throw error;
    }
  }
}
```

### **Rate Limiting Issues**

#### **Too Many Requests**

**Problem**: API returns rate limit errors.

```json
{
  "error": "Too many requests",
  "code": 429,
  "retryAfter": 60
}
```

**Solutions**:
```javascript
// Implement exponential backoff
async function apiCallWithBackoff(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      
      if (response.status === 429) {
        const retryAfter = parseInt(response.headers.get('Retry-After') || '1');
        const delay = Math.min(retryAfter * 1000, 60000); // Max 1 minute
        
        console.warn(`Rate limited, waiting ${delay}ms before retry ${i + 1}`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      
      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000; // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Request queuing for high-volume applications
class RequestQueue {
  constructor(maxConcurrent = 5, delayBetweenRequests = 100) {
    this.queue = [];
    this.running = 0;
    this.maxConcurrent = maxConcurrent;
    this.delay = delayBetweenRequests;
  }
  
  async add(requestFn) {
    return new Promise((resolve, reject) => {
      this.queue.push({ requestFn, resolve, reject });
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.maxConcurrent || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { requestFn, resolve, reject } = this.queue.shift();
    
    try {
      const result = await requestFn();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      setTimeout(() => this.process(), this.delay);
    }
  }
}
```

---

## **Performance Issues**

### **Slow Response Times**

#### **Database Query Optimization**

**Problem**: API responses are slow due to inefficient queries.

**Diagnosis**:
```javascript
// Add query timing
const startTime = Date.now();
const result = await database.query(sql, params);
const duration = Date.now() - startTime;

if (duration > 1000) {
  console.warn(`Slow query (${duration}ms):`, sql);
}
```

**Solutions**:
```javascript
// Implement query caching
const cache = new Map();

async function cachedQuery(sql, params, ttl = 300000) { // 5 minutes
  const key = `${sql}:${JSON.stringify(params)}`;
  const cached = cache.get(key);
  
  if (cached && Date.now() - cached.timestamp < ttl) {
    return cached.result;
  }
  
  const result = await database.query(sql, params);
  cache.set(key, { result, timestamp: Date.now() });
  
  return result;
}

// Add database connection pooling
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});
```

#### **Memory Leaks**

**Problem**: Application memory usage grows over time.

**Diagnosis**:
```javascript
// Monitor memory usage
setInterval(() => {
  const usage = process.memoryUsage();
  console.log('Memory usage:', {
    rss: `${Math.round(usage.rss / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)}MB`,
    heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)}MB`,
    external: `${Math.round(usage.external / 1024 / 1024)}MB`
  });
}, 30000);
```

**Solutions**:
```javascript
// Proper cleanup of resources
class ResourceManager {
  constructor() {
    this.resources = new Set();
  }
  
  add(resource) {
    this.resources.add(resource);
    return resource;
  }
  
  cleanup() {
    for (const resource of this.resources) {
      if (resource.close) resource.close();
      if (resource.destroy) resource.destroy();
      if (resource.end) resource.end();
    }
    this.resources.clear();
  }
}

// Use WeakMap for caching to prevent memory leaks
const cache = new WeakMap();

// Implement proper error handling to prevent resource leaks
async function safeOperation(operation) {
  const resources = new ResourceManager();
  
  try {
    return await operation(resources);
  } finally {
    resources.cleanup();
  }
}
```

---

## **Production Deployment Issues**

### **Docker Problems**

#### **Container Build Failures**

**Problem**: Docker build fails with various errors.

```bash
Error: failed to solve: process "/bin/sh -c npm install" did not complete successfully
```

**Solutions**:
```dockerfile
# Use multi-stage build to reduce image size
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Add proper health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1
```

#### **Container Runtime Issues**

**Problem**: Container starts but application fails.

**Diagnosis**:
```bash
# Check container logs
docker logs container-name

# Inspect container
docker exec -it container-name sh

# Check environment variables
docker exec container-name env
```

**Solutions**:
```dockerfile
# Add proper signal handling
FROM node:18-alpine
RUN apk add --no-cache dumb-init
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]

# Use non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs
```

### **Load Balancer Issues**

#### **Health Check Failures**

**Problem**: Load balancer marks instances as unhealthy.

**Solutions**:
```javascript
// Comprehensive health check endpoint
app.get('/health', async (req, res) => {
  const health = {
    status: 'OK',
    timestamp: new Date().toISOString(),
    checks: {}
  };
  
  try {
    // Check database connection
    await database.query('SELECT 1');
    health.checks.database = 'OK';
    
    // Check Stellar connection
    const server = new Server(process.env.STELLAR_HORIZON_URL);
    await server.ledgers().limit(1).call();
    health.checks.stellar = 'OK';
    
    // Check memory usage
    const memUsage = process.memoryUsage();
    health.checks.memory = {
      status: memUsage.heapUsed < 500 * 1024 * 1024 ? 'OK' : 'WARNING',
      heapUsed: Math.round(memUsage.heapUsed / 1024 / 1024)
    };
    
    res.json(health);
  } catch (error) {
    health.status = 'ERROR';
    health.error = error.message;
    res.status(503).json(health);
  }
});

// Separate readiness check
app.get('/ready', (req, res) => {
  // Check if application is ready to serve requests
  if (applicationReady) {
    res.json({ status: 'ready' });
  } else {
    res.status(503).json({ status: 'not ready' });
  }
});
```

---

## **Monitoring and Debugging**

### **Logging Best Practices**

```javascript
// Structured logging with correlation IDs
const winston = require('winston');
const { v4: uuidv4 } = require('uuid');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' })
  ]
});

// Add correlation ID middleware
app.use((req, res, next) => {
  req.correlationId = uuidv4();
  res.setHeader('X-Correlation-ID', req.correlationId);
  
  logger.info('Request started', {
    correlationId: req.correlationId,
    method: req.method,
    url: req.url,
    userAgent: req.get('User-Agent')
  });
  
  next();
});
```

### **Error Tracking**

```javascript
// Centralized error handling
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

// Global error handler
app.use((error, req, res, next) => {
  logger.error('Unhandled error', {
    correlationId: req.correlationId,
    error: error.message,
    stack: error.stack,
    statusCode: error.statusCode
  });
  
  if (error.isOperational) {
    res.status(error.statusCode).json({
      error: error.message,
      code: error.code,
      correlationId: req.correlationId
    });
  } else {
    res.status(500).json({
      error: 'Internal server error',
      correlationId: req.correlationId
    });
  }
});
```

---

## **Getting Help**

### **Debug Information Collection**

When reporting issues, please include:

```bash
# System information
node --version
npm --version
docker --version

# Environment details
echo $NODE_ENV
echo $STELLAR_NETWORK

# Application logs
tail -n 100 app.log

# Network connectivity
curl -I https://horizon-testnet.stellar.org/
```

### **Common Debug Commands**

```bash
# Check API health
curl -X GET http://localhost:3000/health

# Test specific endpoint
curl -X POST http://localhost:3000/api/credentials \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{"data": "test"}'

# Monitor real-time logs
docker logs -f container-name

# Check resource usage
docker stats container-name
```

### **Support Channels**

- **GitHub Issues**: Report bugs and feature requests
- **Documentation**: Check the latest documentation
- **Community Forum**: Ask questions and share solutions
- **Discord/Slack**: Real-time community support

---

This troubleshooting guide covers the most common issues you might encounter. If you're still experiencing problems, please provide detailed error messages, logs, and system information when seeking help.

*Previous: [Deployment Guide](./08-deployment.md) | Next: [API Reference](./api-reference.md)*