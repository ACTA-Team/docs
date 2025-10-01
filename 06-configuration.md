# Configuration

## Overview

The ACTA API uses environment variables for configuration management, ensuring secure and flexible deployment across different environments. This section covers all configuration options and best practices.

---

## **Environment Variables**

### **Required Variables**

These variables must be set for the API to function properly:

| Variable | Description | Example |
|----------|-------------|---------|
| `STELLAR_SECRET_KEY` | Stellar account secret key | `SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` |
| `STELLAR_HORIZON_URL` | Horizon server URL | `https://horizon-testnet.stellar.org` |

### **Optional Variables**

These variables have default values but can be customized:

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `PORT` | Server port | `8000` | `3000` |
| `NODE_ENV` | Environment mode | `development` | `production` |
| `STELLAR_SOROBAN_URL` | Soroban RPC URL | `https://soroban-testnet.stellar.org` | Custom URL |
| `STELLAR_NETWORK_PASSPHRASE` | Network identifier | `Test SDF Network ; September 2015` | Custom passphrase |

---

## **Environment Setup**

### **Development Environment**

Create a `.env` file in your project root:

```bash
# Stellar Configuration
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_SOROBAN_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK_PASSPHRASE=Test SDF Network ; September 2015

# Server Configuration
PORT=8000
NODE_ENV=development

# Logging
LOG_LEVEL=debug
```

### **Production Environment**

For production deployments, set environment variables through your hosting platform:

```bash
# Railway.app example
railway variables set STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
railway variables set STELLAR_HORIZON_URL=https://horizon.stellar.org
railway variables set NODE_ENV=production
```

### **Testing Environment**

For testing, use a separate configuration:

```bash
# .env.test
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
NODE_ENV=test
PORT=8001
```

---

## **Configuration Validation**

The API validates required environment variables on startup:

```javascript
const requiredEnvVars = [
  'STELLAR_SECRET_KEY',
  'STELLAR_HORIZON_URL'
];

const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  console.error(`Missing required environment variables: ${missingVars.join(', ')}`);
  process.exit(1);
}
```

---

## **Stellar Network Configuration**

### **Testnet Configuration**

For development and testing:

```bash
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_SOROBAN_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK_PASSPHRASE=Test SDF Network ; September 2015
```

**Features:**
- Free XLM from friendbot
- No real value transactions
- Faster transaction confirmation
- Reset periodically

### **Mainnet Configuration**

For production use:

```bash
STELLAR_HORIZON_URL=https://horizon.stellar.org
STELLAR_SOROBAN_URL=https://soroban-mainnet.stellar.org
STELLAR_NETWORK_PASSPHRASE=Public Global Stellar Network ; September 2015
```

**Requirements:**
- Real XLM for transactions
- Higher security standards
- Production-grade monitoring

---

## **Security Best Practices**

### **Secret Key Management**

**Do:**
- Store secret keys in environment variables
- Use different keys for different environments
- Rotate keys regularly
- Use key management services in production

**Don't:**
- Hardcode keys in source code
- Commit keys to version control
- Share keys in plain text
- Use production keys in development

### **Environment Separation**

```bash
# Development
STELLAR_SECRET_KEY=SDEV_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Staging
STELLAR_SECRET_KEY=SSTG_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Production
STELLAR_SECRET_KEY=SPRD_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

---

## **Configuration Loading**

### **Environment File Loading**

The API uses `dotenv` to load configuration:

```javascript
// Load environment variables
require('dotenv').config();

// Configuration object
const config = {
  port: process.env.PORT || 8000,
  nodeEnv: process.env.NODE_ENV || 'development',
  stellar: {
    secretKey: process.env.STELLAR_SECRET_KEY,
    horizonUrl: process.env.STELLAR_HORIZON_URL,
    sorobanUrl: process.env.STELLAR_SOROBAN_URL || 'https://soroban-testnet.stellar.org',
    networkPassphrase: process.env.STELLAR_NETWORK_PASSPHRASE || 'Test SDF Network ; September 2015'
  }
};
```

### **Configuration Validation**

```javascript
function validateConfig(config) {
  const errors = [];

  if (!config.stellar.secretKey) {
    errors.push('STELLAR_SECRET_KEY is required');
  }

  if (!config.stellar.horizonUrl) {
    errors.push('STELLAR_HORIZON_URL is required');
  }

  if (errors.length > 0) {
    throw new Error(`Configuration errors: ${errors.join(', ')}`);
  }

  return config;
}
```

---

## **Deployment Configurations**

### **Railway.app**

Railway automatically detects Node.js applications. Set environment variables in the dashboard:

1. Go to your project dashboard
2. Click on "Variables" tab
3. Add required environment variables
4. Deploy your application

### **Heroku**

Configure environment variables using the Heroku CLI:

```bash
heroku config:set STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
heroku config:set STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
heroku config:set NODE_ENV=production
```

### **Kubernetes**

Use ConfigMaps and Secrets for configuration:

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: acta-config
data:
  STELLAR_HORIZON_URL: "https://horizon-testnet.stellar.org"
  NODE_ENV: "production"
  PORT: "8000"
```

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: acta-secrets
type: Opaque
data:
  STELLAR_SECRET_KEY: <base64-encoded-secret-key>
```

---

## **Configuration Monitoring**

### **Health Checks**

Monitor configuration health:

```javascript
app.get('/health', async (req, res) => {
  const health = {
    status: 'OK',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV,
    stellar: {
      network: process.env.STELLAR_NETWORK_PASSPHRASE?.includes('Test') ? 'testnet' : 'mainnet',
      horizonUrl: process.env.STELLAR_HORIZON_URL
    }
  };

  res.json(health);
});
```

### **Configuration Logging**

Log configuration on startup (without sensitive data):

```javascript
console.log('Starting ACTA API with configuration:');
console.log(`- Environment: ${process.env.NODE_ENV}`);
console.log(`- Port: ${process.env.PORT || 8000}`);
console.log(`- Stellar Network: ${process.env.STELLAR_NETWORK_PASSPHRASE?.includes('Test') ? 'Testnet' : 'Mainnet'}`);
console.log(`- Horizon URL: ${process.env.STELLAR_HORIZON_URL}`);
```

---

## **Troubleshooting**

### **Common Configuration Issues**

#### Missing Environment Variables

**Error:**
```
Missing required environment variables: STELLAR_SECRET_KEY
```

**Solution:**
1. Check if `.env` file exists
2. Verify variable names are correct
3. Ensure no extra spaces in variable definitions

#### Invalid Secret Key Format

**Error:**
```
Invalid Stellar secret key format
```

**Solution:**
1. Verify the secret key starts with 'S'
2. Check the key is 56 characters long
3. Ensure no extra characters or spaces

#### Network Connection Issues

**Error:**
```
Failed to connect to Stellar network
```

**Solution:**
1. Verify Horizon URL is accessible
2. Check network connectivity
3. Confirm the correct network (testnet/mainnet)

### **Configuration Debugging**

Enable debug logging to troubleshoot configuration issues:

```bash
NODE_ENV=development LOG_LEVEL=debug npm start
```

---

## **Best Practices**

### **Environment Management**

1. **Use separate configurations** for each environment
2. **Document all variables** and their purposes
3. **Validate configuration** on application startup
4. **Use default values** where appropriate
5. **Monitor configuration changes** in production

### **Security Guidelines**

1. **Never commit secrets** to version control
2. **Use environment-specific keys** for different stages
3. **Implement key rotation** policies
4. **Monitor access** to configuration systems
5. **Use encrypted storage** for sensitive configuration

### **Deployment Checklist**

Before deploying to production:

- [ ] All required environment variables are set
- [ ] Secret keys are production-grade
- [ ] Network configuration matches target environment
- [ ] Configuration validation passes
- [ ] Health checks are working
- [ ] Monitoring is configured
- [ ] Backup procedures are in place

---

*Next: Learn about [Examples and Use Cases](./07-examples.md) to see practical implementations of the API.*