# Configuration

## Overview

The ACTA API uses environment variables for configuration management, ensuring secure and flexible deployment across different environments. This section covers all configuration options and best practices.

---

## **Environment Variables**

### **Required Variables**

Estas variables deben estar configuradas para que el API funcione correctamente:

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `STELLAR_SECRET_KEY` | Clave secreta de la cuenta Stellar usada por el servidor | `SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` |
| `ACTA_ISSUANCE_CONTRACT_ID` | ID del contrato de emisión (Soroban) | `CC5VJEF2D56G5C3JL6JKGS5T3NNHTE7LCLSK26CMX55P55F3GP365KN6` |
| `ACTA_VAULT_CONTRACT_ID` | ID del contrato de vault (Soroban) | `CCC3DH5D3P2VP2OXCFYSC25E6I7SKHKPH3GLISJIR3IMZ3W2STKTESIE` |
| `ACTA_DID_CONTRACT_ID` | ID del contrato DID (Soroban) | `CBD2UKHTCYDHDGKLFZ47NOFXSFVTWBA6GADR5EYHPIHGXBW6RKOGTB4V` |

### **Optional Variables**

Estas variables tienen valores por defecto pero pueden ser personalizadas:

| Variable | Descripción | Por defecto | Ejemplo |
|----------|-------------|------------|---------|
| `PORT` | Puerto del servidor | `8000` | `3000` |
| `NODE_ENV` | Modo de ejecución | `development` | `production` |
| `STELLAR_NETWORK` | Red Stellar | `testnet` | `mainnet` |
| `STELLAR_HORIZON_URL` | URL de Horizon | según red | `https://horizon-testnet.stellar.org` |
| `SOROBAN_RPC_URL` | URL de Soroban RPC | según red | `https://soroban-testnet.stellar.org` |
| `ACTA_DEPLOYER_CONTRACT_ID` | ID opcional del contrato deployer | — | `CDIBXZ...` |

---

## **Environment Setup**

### **Development Environment**

Crea un archivo `.env` en el root del proyecto (valores de ejemplo para Testnet):

```bash
# Stellar Configuration
STELLAR_NETWORK=testnet
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org

# Server Configuration
PORT=8000
NODE_ENV=development

# ACTA Contracts
ACTA_ISSUANCE_CONTRACT_ID=CC5VJEF2D56G5C3JL6JKGS5T3NNHTE7LCLSK26CMX55P55F3GP365KN6
ACTA_VAULT_CONTRACT_ID=CCC3DH5D3P2VP2OXCFYSC25E6I7SKHKPH3GLISJIR3IMZ3W2STKTESIE
ACTA_DID_CONTRACT_ID=CBD2UKHTCYDHDGKLFZ47NOFXSFVTWBA6GADR5EYHPIHGXBW6RKOGTB4V
# ACTA_DEPLOYER_CONTRACT_ID=CDIBXZUIZBQG7IMFFCMTSIKTL6JCPSVENHHZQ6CUXD5E26WNF6PVVQLK
```

### **Production Environment**

Para despliegues en producción, configura las variables de entorno en tu plataforma de hosting:

```bash
# Railway.app example
railway variables set STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
railway variables set STELLAR_NETWORK=mainnet
railway variables set STELLAR_HORIZON_URL=https://horizon.stellar.org
railway variables set SOROBAN_RPC_URL=https://soroban-mainnet.stellar.org
railway variables set ACTA_ISSUANCE_CONTRACT_ID=...
railway variables set ACTA_VAULT_CONTRACT_ID=...
railway variables set ACTA_DID_CONTRACT_ID=...
railway variables set NODE_ENV=production
```

### **Testing Environment**

For testing, use a separate configuration:

```bash
# .env.test
STELLAR_NETWORK=testnet
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
ACTA_ISSUANCE_CONTRACT_ID=...
ACTA_VAULT_CONTRACT_ID=...
ACTA_DID_CONTRACT_ID=...
NODE_ENV=test
PORT=8001
```

---

## **Configuration Validation**

El API valida las variables requeridas al iniciar:

```javascript
const requiredEnvVars = [
  'STELLAR_SECRET_KEY',
  'ACTA_ISSUANCE_CONTRACT_ID',
  'ACTA_VAULT_CONTRACT_ID',
  'ACTA_DID_CONTRACT_ID'
];

const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  console.error(`Missing required environment variables: ${missingVars.join(', ')}`);
  process.exit(1);
}
```

---

## **Configuración de Red Stellar**

### **Testnet Configuration**

Para desarrollo y pruebas:

```bash
STELLAR_NETWORK=testnet
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
```

**Features:**
- Free XLM from friendbot
- No real value transactions
- Faster transaction confirmation
- Reset periodically

### **Mainnet Configuration**

Para uso en producción:

```bash
STELLAR_NETWORK=mainnet
STELLAR_HORIZON_URL=https://horizon.stellar.org
SOROBAN_RPC_URL=https://soroban-mainnet.stellar.org
```

**Requirements:**
- Real XLM for transactions
- Higher security standards
- Production-grade monitoring

---

## **Buenas Prácticas de Seguridad**

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

## **Carga de Configuración**

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
    sorobanRpcUrl: process.env.SOROBAN_RPC_URL || 'https://soroban-testnet.stellar.org',
    networkPassphrase: (process.env.STELLAR_NETWORK || 'testnet') === 'mainnet'
      ? 'Public Global Stellar Network ; September 2015'
      : 'Test SDF Network ; September 2015'
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

  // horizonUrl y sorobanRpcUrl tienen valores por defecto según la red

  if (errors.length > 0) {
    throw new Error(`Configuration errors: ${errors.join(', ')}`);
  }

  return config;
}
```

---

## **Configuraciones de Despliegue**

### **Railway.app**

Railway automatically detects Node.js applications. Set environment variables in the dashboard:

1. Go to your project dashboard
2. Click on "Variables" tab
3. Añade las variables de entorno requeridas

---

## **Prerequisitos de Contratos (ACTA)**

- `STELLAR_SECRET_KEY` debe pertenecer al administrador del contrato de emisión (o ejecútese `set_admin`).
- Autoriza al emisor en el `VaultContract` usando la clave pública de `STELLAR_SECRET_KEY` (`authorize_issuer`).
- Asegura saldo suficiente de XLM en la cuenta según la red seleccionada.
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