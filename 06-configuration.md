# Configuration and Environment Variables

## Overview

The ACTA API uses environment variables for configuration management, allowing for flexible deployment across different environments (development, staging, production) without code changes. This approach follows the [12-Factor App](https://12factor.net/config) methodology for configuration management.

## Environment Variables

### Required Variables

These environment variables must be set for the API to function properly:

#### STELLAR_SECRET_KEY
- **Type**: String (56 characters, starts with 'S')
- **Description**: The Stellar secret key used for signing transactions
- **Example**: `SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`
- **Security**: Keep this secret and never commit to version control

#### STELLAR_HORIZON_URL
- **Type**: URL
- **Description**: The Horizon server URL for Stellar network operations
- **Development**: `https://horizon-testnet.stellar.org`
- **Production**: `https://horizon.stellar.org`

### Optional Variables

These variables have default values but can be customized:

#### STELLAR_SOROBAN_URL
- **Type**: URL
- **Description**: The Soroban RPC server URL for smart contract operations
- **Default**: `https://soroban-testnet.stellar.org`
- **Production**: `https://soroban.stellar.org`

#### STELLAR_NETWORK_PASSPHRASE
- **Type**: String
- **Description**: The network passphrase for transaction signing
- **Default**: `Test SDF Network ; September 2015` (Testnet)
- **Production**: `Public Global Stellar Network ; September 2015`

#### PORT
- **Type**: Number
- **Description**: The port number for the API server
- **Default**: `3000`
- **Example**: `8080`

#### NODE_ENV
- **Type**: String
- **Description**: The application environment
- **Values**: `development`, `staging`, `production`
- **Default**: `development`

## Environment Configuration Files

### .env.example

The repository includes a `.env.example` file that serves as a template:

```bash
# Stellar Network Configuration
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_SOROBAN_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK_PASSPHRASE=Test SDF Network ; September 2015

# Server Configuration
PORT=3000
NODE_ENV=development

# Optional: Logging Configuration
LOG_LEVEL=info
```

### Creating Your .env File

1. Copy the example file:
```bash
cp .env.example .env
```

2. Update the values with your actual configuration:
```bash
# Edit the .env file with your preferred editor
nano .env
```

3. Ensure the file is not tracked by Git:
```bash
# .env should already be in .gitignore
echo ".env" >> .gitignore
```

## Environment-Specific Configurations

### Development Environment

```bash
# .env.development
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_SOROBAN_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK_PASSPHRASE=Test SDF Network ; September 2015
PORT=3000
NODE_ENV=development
LOG_LEVEL=debug
```

### Staging Environment

```bash
# .env.staging
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_SOROBAN_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK_PASSPHRASE=Test SDF Network ; September 2015
PORT=3000
NODE_ENV=staging
LOG_LEVEL=info
```

### Production Environment

```bash
# .env.production
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon.stellar.org
STELLAR_SOROBAN_URL=https://soroban.stellar.org
STELLAR_NETWORK_PASSPHRASE=Public Global Stellar Network ; September 2015
PORT=80
NODE_ENV=production
LOG_LEVEL=warn
```

## Configuration Validation

The API performs validation of environment variables at startup:

### Required Variable Check

```typescript
const requiredEnvVars = [
  'STELLAR_SECRET_KEY',
  'STELLAR_HORIZON_URL'
];

const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  throw new Error(`Missing required environment variables: ${missingVars.join(', ')}`);
}
```

### Format Validation

```typescript
// Validate Stellar secret key format
const secretKey = process.env.STELLAR_SECRET_KEY;
if (!secretKey || secretKey.length !== 56 || !secretKey.startsWith('S')) {
  throw new Error('Invalid STELLAR_SECRET_KEY format');
}

// Validate URL format
const horizonUrl = process.env.STELLAR_HORIZON_URL;
try {
  new URL(horizonUrl);
} catch (error) {
  throw new Error('Invalid STELLAR_HORIZON_URL format');
}
```

## CORS Configuration

The API includes configurable CORS settings for cross-origin requests:

### Default CORS Settings

```typescript
// src/config/cors.ts
const corsOptions = {
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  methods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
};
```

### Environment Variables for CORS

#### ALLOWED_ORIGINS
- **Type**: Comma-separated URLs
- **Description**: Origins allowed to make requests to the API
- **Example**: `http://localhost:3000,https://app.yourdomain.com`
- **Default**: `http://localhost:3000`

## Security Headers Configuration

The API includes security headers that can be configured:

```typescript
// Security headers configuration
const securityHeaders = {
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Strict-Transport-Security': process.env.NODE_ENV === 'production' 
    ? 'max-age=31536000; includeSubDomains' 
    : undefined
};
```

## Logging Configuration

### LOG_LEVEL
- **Type**: String
- **Description**: Minimum log level to output
- **Values**: `error`, `warn`, `info`, `debug`
- **Default**: `info`

### Log Format

The API uses structured logging with different formats for different environments:

```typescript
const logFormat = process.env.NODE_ENV === 'production' 
  ? 'combined'  // Apache combined log format
  : 'dev';      // Colored output for development
```

## Docker Configuration

When using Docker, environment variables can be passed through various methods:

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  acta-api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - STELLAR_SECRET_KEY=${STELLAR_SECRET_KEY}
      - STELLAR_HORIZON_URL=${STELLAR_HORIZON_URL}
      - STELLAR_SOROBAN_URL=${STELLAR_SOROBAN_URL}
      - NODE_ENV=production
    env_file:
      - .env
```

### Docker Run

```bash
docker run -d \
  --name acta-api \
  -p 3000:3000 \
  -e STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
  -e STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org \
  -e NODE_ENV=production \
  acta-api:latest
```

### Environment File with Docker

```bash
# Use environment file
docker run -d \
  --name acta-api \
  -p 3000:3000 \
  --env-file .env \
  acta-api:latest
```

## Cloud Deployment Configuration

### AWS ECS

```json
{
  "family": "acta-api",
  "containerDefinitions": [
    {
      "name": "acta-api",
      "image": "your-registry/acta-api:latest",
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        },
        {
          "name": "PORT",
          "value": "3000"
        }
      ],
      "secrets": [
        {
          "name": "STELLAR_SECRET_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:stellar-secret-key"
        }
      ]
    }
  ]
}
```

### Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: acta-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: acta-api
  template:
    metadata:
      labels:
        app: acta-api
    spec:
      containers:
      - name: acta-api
        image: your-registry/acta-api:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        - name: STELLAR_HORIZON_URL
          value: "https://horizon.stellar.org"
        envFrom:
        - secretRef:
            name: stellar-secrets
```

### Heroku

```bash
# Set environment variables
heroku config:set STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
heroku config:set STELLAR_HORIZON_URL=https://horizon.stellar.org
heroku config:set NODE_ENV=production
```

## Configuration Best Practices

### Security

1. **Never commit secrets**: Use `.gitignore` to exclude `.env` files
2. **Use secret management**: In production, use dedicated secret management services
3. **Rotate keys regularly**: Implement key rotation policies
4. **Principle of least privilege**: Only provide necessary permissions

### Environment Separation

1. **Separate configurations**: Use different configurations for each environment
2. **Validate early**: Validate configuration at application startup
3. **Document requirements**: Clearly document all required variables
4. **Use defaults wisely**: Provide sensible defaults for optional variables

### Monitoring

1. **Log configuration**: Log (non-sensitive) configuration at startup
2. **Health checks**: Include configuration validation in health checks
3. **Alerts**: Set up alerts for configuration-related failures

## Troubleshooting Configuration Issues

### Common Problems

#### Missing Environment Variables

**Error**: `Missing required environment variables: STELLAR_SECRET_KEY`

**Solution**: Ensure all required environment variables are set in your `.env` file or environment.

#### Invalid Secret Key Format

**Error**: `Invalid STELLAR_SECRET_KEY format`

**Solution**: Verify that your secret key is 56 characters long and starts with 'S'.

#### Network Connection Issues

**Error**: `Failed to connect to Horizon server`

**Solution**: Check that `STELLAR_HORIZON_URL` is correct and accessible from your environment.

#### CORS Issues

**Error**: `CORS policy: No 'Access-Control-Allow-Origin' header`

**Solution**: Add your frontend domain to the `ALLOWED_ORIGINS` environment variable.

### Debugging Configuration

Enable debug logging to troubleshoot configuration issues:

```bash
LOG_LEVEL=debug npm start
```

This will output detailed information about the configuration loading process.

### Configuration Validation Script

Create a script to validate your configuration:

```typescript
// scripts/validate-config.ts
import { config } from 'dotenv';

config();

const requiredVars = [
  'STELLAR_SECRET_KEY',
  'STELLAR_HORIZON_URL'
];

console.log('Validating configuration...');

for (const varName of requiredVars) {
  const value = process.env[varName];
  if (!value) {
    console.error(`❌ Missing: ${varName}`);
    process.exit(1);
  } else {
    console.log(`✅ Found: ${varName}`);
  }
}

console.log('✅ Configuration is valid!');
```

Run the validation script:

```bash
npx ts-node scripts/validate-config.ts
```

---

*Next: Learn about [Examples and Use Cases](./07-examples.md) to see practical implementations of the API.*