# API Integration Setup

## Getting Started with ACTA API

The ACTA API is a hosted service that you can integrate directly into your applications. No installation required!

**Base URL:** `https://api.acta.build`

## Prerequisites

Before integrating with ACTA API, you'll need:

### Required
- **Stellar Account**: A Stellar blockchain account for credential operations
- **Basic Knowledge**: Understanding of REST APIs and HTTP requests
- **Network Access**: Internet connection to reach the ACTA API

### Recommended
- **API Client**: Tools like Postman, curl, or your preferred HTTP client
- **Development Environment**: Node.js, Python, or your preferred programming language

## Quick Integration Steps

### 1. Get Your Stellar Account

For testing and development, create a testnet account:

**Option A: Using Stellar Laboratory (Recommended)**
1. Visit [Stellar Laboratory](https://laboratory.stellar.org/#account-creator)
2. Click "Generate keypair" to create a new account
3. Copy both the **Public Key** and **Secret Key**
4. Click "Fund account" to add test XLM

**Option B: Using Stellar SDK**
```javascript
const StellarSdk = require('stellar-sdk');
const pair = StellarSdk.Keypair.random();

console.log('Public Key:', pair.publicKey());
console.log('Secret Key:', pair.secret());
```

### 2. Test API Connection

Verify the API is accessible:

```bash
# Test API connectivity
curl https://api.acta.build/health

# Expected response:
# {
#   "status": "OK",
#   "timestamp": "2025-01-XX:XX:XX.XXXZ",
#   "service": "Stellar Credential API"
# }
```

### 3. Make Your First API Call

Create your first credential:

```bash
# Using curl (requires API key from apikeys.acta.build)
curl -X POST https://api.acta.build/credentials \
  -H "Content-Type: application/json" \
  -H "X-ACTA-Key: your_api_key_here" \
  -d '{
    "recipient": "YOUR_STELLAR_PUBLIC_KEY",
    "issuer": "YOUR_STELLAR_PUBLIC_KEY",
    "credentialType": "certificate",
    "metadata": {
      "title": "My First Credential",
      "description": "Testing ACTA integration"
    }
  }'
```

**Using JavaScript/Node.js:**
```javascript
const response = await fetch('https://api.acta.build/credentials', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    recipient: "YOUR_STELLAR_PUBLIC_KEY",
    issuer: "YOUR_STELLAR_PUBLIC_KEY",
    credentialType: "certificate",
    metadata: {
      title: "My First Credential",
      description: "Testing ACTA integration"
    }
  })
});

const result = await response.json();
console.log('Credential created:', result);
```

**Using Python:**
```python
import requests

url = "https://api.acta.build/credentials"
data = {
    "recipient": "YOUR_STELLAR_PUBLIC_KEY",
    "issuer": "YOUR_STELLAR_PUBLIC_KEY",
    "credentialType": "certificate",
    "metadata": {
        "title": "My First Credential",
        "description": "Testing ACTA integration"
    }
}

response = requests.post(url, json=data)
result = response.json()
print("Credential created:", result)
```

## Integration Examples

### Web Application Integration

For web applications, you can integrate ACTA API using standard HTTP requests:

```html
<!DOCTYPE html>
<html>
<head>
    <title>ACTA Integration Example</title>
</head>
<body>
    <button onclick="createCredential()">Create Credential</button>
    
    <script>
    async function createCredential() {
        try {
            const response = await fetch('https://api.acta.build/credentials', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    recipient: "YOUR_STELLAR_PUBLIC_KEY",
                    issuer: "YOUR_STELLAR_PUBLIC_KEY",
                    credentialType: "certificate",
                    metadata: {
                        title: "Web App Credential",
                        description: "Created from web application"
                    }
                })
            });
            
            const result = await response.json();
            console.log('Success:', result);
            alert('Credential created successfully!');
        } catch (error) {
            console.error('Error:', error);
            alert('Failed to create credential');
        }
    }
    </script>
</body>
</html>
```

### Mobile App Integration

For mobile applications, use your platform's HTTP client:

**React Native:**
```javascript
import React from 'react';
import { Button, Alert } from 'react-native';

const createCredential = async () => {
  try {
    const response = await fetch('https://api.acta.build/credentials', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        recipient: "YOUR_STELLAR_PUBLIC_KEY",
        issuer: "YOUR_STELLAR_PUBLIC_KEY",
        credentialType: "certificate",
        metadata: {
          title: "Mobile App Credential",
          description: "Created from mobile app"
        }
      }),
    });
    
    const result = await response.json();
    Alert.alert('Success', 'Credential created successfully!');
  } catch (error) {
    Alert.alert('Error', 'Failed to create credential');
  }
};

export const CredentialButton = () => (
  <Button title="Create Credential" onPress={createCredential} />
);
```

## Next Steps

1. **Explore API Endpoints**: Check the [API Endpoints](./04-endpoints.md) documentation for all available operations
2. **Review Examples**: See [Examples and Use Cases](./07-examples.md) for more integration patterns
3. **Security**: Read [Authentication and Security](./03-authentication.md) for production considerations
4. **Troubleshooting**: Visit [Troubleshooting](./09-troubleshooting.md) if you encounter issues

## Need to Run Your Own Instance?

If you need to run your own instance of ACTA API (for development, customization, or self-hosting), see the [Contributors Guide](./10-contributors.md) for complete setup instructions.

## Troubleshooting

### Common Issues

#### 1. Stellar Connection Issues

**Error**: `Failed to connect to Stellar network`

**Solutions**:
- Check your internet connection
- Verify the `STELLAR_HORIZON_URL` is correct
- Ensure the Stellar network is not experiencing downtime

#### 2. Invalid Secret Key

**Error**: `Invalid STELLAR_SECRET_KEY format`

**Solutions**:
- Ensure the secret key starts with 'S'
- Verify the secret key is 56 characters long
- Check for any extra spaces or characters

#### 3. Insufficient Balance

**Error**: `Insufficient XLM balance for contract deployment`

**Solutions**:
- Fund your Stellar account with at least 5 XLM
- For testnet, use the [Stellar Laboratory](https://laboratory.stellar.org/#account-creator) to fund your account
- For mainnet, purchase XLM from a cryptocurrency exchange

#### 4. Port Already in Use

**Error**: `EADDRINUSE: address already in use :::3000`

**Solutions**:
- Change the `PORT` in your `.env` file
- Kill the process using the port: `lsof -ti:3000 | xargs kill -9`
- Use a different port: `PORT=3001 npm run dev`

### Getting Help

If you encounter issues not covered here:

1. Review the application logs for detailed error messages
2. Consult the [Stellar Developer Documentation](https://developers.stellar.org/)

---

*Next: Learn about [Authentication and Security](./03-authentication.md) to understand how the API protects your credentials.*