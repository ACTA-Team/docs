# Authentication and Security

## Security Overview

The ACTA API implements multiple layers of security to protect credential data and ensure secure communication between clients and the Stellar blockchain. This chapter covers all security mechanisms implemented in the API.

## Security Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client        │    │   ACTA API      │    │ Stellar Network │
│                 │    │                 │    │                 │
│ • HTTPS Only    │◄──►│ • CORS Headers  │◄──►│ • Cryptographic │
│ • API Keys      │    │ • Security Hdrs │    │   Signatures    │
│ • Input Valid.  │    │ • Input Valid.  │    │ • Immutable     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## CORS Configuration

The API implements comprehensive Cross-Origin Resource Sharing (CORS) protection to control which domains can access the API.

### CORS Middleware Implementation

```typescript
// src/config/cors.ts
export const corsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization, X-ACTA-Key');
  res.header('Access-Control-Max-Age', '86400');

  if (req.method === 'OPTIONS') {
    res.status(200).end();
    return;
  }

  next();
};
```

### CORS Configuration Options

#### Development Configuration
```env
# Allow all origins for development
ALLOWED_ORIGINS=*
```

#### Production Configuration
```env
# Restrict to specific domains
ALLOWED_ORIGINS=https://yourdomain.com,https://app.yourdomain.com,https://admin.yourdomain.com
```

### Customizing CORS

To customize CORS for your specific needs, modify the `corsMiddleware` function:

```typescript
export const corsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || ['*'];
  const origin = req.headers.origin;

  if (allowedOrigins.includes('*') || (origin && allowedOrigins.includes(origin))) {
    res.header('Access-Control-Allow-Origin', origin || '*');
  }

  // Rest of the configuration...
};
```

## Security Headers

The API implements comprehensive HTTP security headers using Helmet.js and custom middleware.

### Helmet.js Configuration

```typescript
// Automatic security headers via Helmet
app.use(helmet());
```

Helmet automatically adds:
- `X-DNS-Prefetch-Control`: Controls DNS prefetching
- `X-Frame-Options`: Prevents clickjacking attacks
- `X-Powered-By`: Removes the X-Powered-By header
- `Strict-Transport-Security`: Enforces HTTPS connections
- `X-Download-Options`: Prevents IE from executing downloads
- `X-Content-Type-Options`: Prevents MIME type sniffing
- `X-XSS-Protection`: Enables XSS filtering

### Custom Security Headers

```typescript
export const securityHeaders = (req: Request, res: Response, next: NextFunction) => {
  res.header('X-Content-Type-Options', 'nosniff');
  res.header('X-Frame-Options', 'DENY');
  res.header('X-XSS-Protection', '1; mode=block');
  res.header('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.header('Content-Security-Policy', "default-src 'self'");

  next();
};
```

### Security Headers Explained

| Header | Purpose | Value |
|--------|---------|-------|
| `X-Content-Type-Options` | Prevents MIME type sniffing | `nosniff` |
| `X-Frame-Options` | Prevents clickjacking | `DENY` |
| `X-XSS-Protection` | Enables XSS filtering | `1; mode=block` |
| `Referrer-Policy` | Controls referrer information | `strict-origin-when-cross-origin` |
| `Content-Security-Policy` | Prevents code injection | `default-src 'self'` |

## Stellar Key Authentication

The API uses Stellar cryptographic keys for authentication and authorization.

### Key Validation Process

```typescript
function getStellarService(): StellarService {
  if (!stellarService) {
    // Validate required environment variables
    const secretKey = process.env.STELLAR_SECRET_KEY;
    if (!secretKey) {
      throw new Error('STELLAR_SECRET_KEY environment variable is required');
    }

    // Validate secret key format
    try {
      const keypair = require('@stellar/stellar-sdk').Keypair.fromSecret(secretKey);
      // Stellar account validated successfully
      // Public key: ${keypair.publicKey()}
    } catch (keyError) {
      throw new Error('Invalid STELLAR_SECRET_KEY format. Must be a valid Stellar secret key starting with S');
    }

    // Initialize service with validated key
    stellarService = new StellarService(config);
  }
  return stellarService;
}
```

### Key Security Best Practices

#### 1. Environment Variables
```bash
# Good: Store in environment variables
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Bad: Never hardcode in source code
const secretKey = "SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX";
```

#### 2. Key Rotation
```bash
# Regularly rotate your Stellar keys
# 1. Generate new keypair
# 2. Fund new account
# 3. Update environment variables
# 4. Deploy new configuration
# 5. Revoke old key access
```

#### 3. Access Control
```bash
# Restrict file permissions
chmod 600 .env

# Use secrets management in production
# - AWS Secrets Manager
# - HashiCorp Vault
# - Kubernetes Secrets
```

## Input Validation

The API implements comprehensive input validation to prevent injection attacks and ensure data integrity.

### Request Body Validation

```typescript
router.post('/', async (req: Request, res: Response) => {
  try {
    const credentialRequest: CredentialRequest = req.body;

    // Validate required fields
    if (!credentialRequest.data) {
      return res.status(400).json({
        error: 'Missing required field: data'
      });
    }

    // Validate data structure
    if (typeof credentialRequest.data !== 'object') {
      return res.status(400).json({
        error: 'Invalid data format: must be an object'
      });
    }

    // Process the request...
  } catch (error) {
    // Handle validation errors...
  }
});
```

### Parameter Validation

```typescript
router.get('/:contractId', async (req: Request, res: Response) => {
  try {
    const { contractId } = req.params;

    // Validate contract ID format
    if (!contractId || typeof contractId !== 'string') {
      return res.status(400).json({
        error: 'Contract ID is required and must be a string'
      });
    }

    // Validate contract ID pattern (Stellar contract address format)
    const contractIdPattern = /^C[A-Z0-9]{55}$/;
    if (!contractIdPattern.test(contractId)) {
      return res.status(400).json({
        error: 'Invalid contract ID format'
      });
    }

    // Process the request...
  } catch (error) {
    // Handle validation errors...
  }
});
```

### Data Sanitization

```typescript
// Utility function for data sanitization
export function sanitizeInput(input: any): any {
  if (typeof input === 'string') {
    // Remove potentially dangerous characters
    return input.replace(/[<>\"']/g, '');
  }
  
  if (typeof input === 'object' && input !== null) {
    const sanitized: any = {};
    for (const [key, value] of Object.entries(input)) {
      sanitized[sanitizeInput(key)] = sanitizeInput(value);
    }
    return sanitized;
  }
  
  return input;
}
```

## Error Handling Security

The API implements secure error handling to prevent information leakage while providing useful debugging information.

### Secure Error Responses

```typescript
// Good: Secure error handling
app.use((error: Error, req: express.Request, res: express.Response, next: express.NextFunction) => {
  // Log error for debugging (use proper logging service in production)
  // Error: ${error.message}
  
  // Don't expose internal error details in production
  const isDevelopment = process.env.NODE_ENV === 'development';
  
  res.status(500).json({
    error: 'Internal server error',
    ...(isDevelopment && { details: error.message, stack: error.stack })
  });
});

// Bad: Exposing sensitive information
app.use((error: Error, req: express.Request, res: express.Response, next: express.NextFunction) => {
  res.status(500).json({
    error: error.message,
    stack: error.stack,
    env: process.env // Never expose environment variables!
  });
});
```

### Error Classification

```typescript
export enum ErrorType {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  AUTHENTICATION_ERROR = 'AUTHENTICATION_ERROR',
  AUTHORIZATION_ERROR = 'AUTHORIZATION_ERROR',
  STELLAR_ERROR = 'STELLAR_ERROR',
  INTERNAL_ERROR = 'INTERNAL_ERROR'
}

export class APIError extends Error {
  constructor(
    public type: ErrorType,
    public message: string,
    public statusCode: number = 500,
    public details?: any
  ) {
    super(message);
    this.name = 'APIError';
  }
}
```

## Rate Limiting

Implement rate limiting to prevent abuse and ensure fair usage of the API.

### Basic Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

// Create rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: {
    error: 'Too many requests from this IP, please try again later.',
    retryAfter: '15 minutes'
  },
  standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
  legacyHeaders: false, // Disable the `X-RateLimit-*` headers
});

// Apply rate limiting to all requests
app.use(limiter);
```

### Endpoint-Specific Rate Limiting

```typescript
// Stricter rate limiting for credential creation
const createCredentialLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 5, // Limit to 5 credential creations per minute
  message: {
    error: 'Too many credential creation requests, please try again later.',
    retryAfter: '1 minute'
  }
});

// Apply to credential creation endpoint
app.use('/v1/credentials', createCredentialLimiter);
```

## HTTPS Configuration

Always use HTTPS in production to encrypt data in transit.

### Express HTTPS Setup

```typescript
import https from 'https';
import fs from 'fs';

// HTTPS configuration for production
if (process.env.NODE_ENV === 'production') {
  const options = {
    key: fs.readFileSync('path/to/private-key.pem'),
    cert: fs.readFileSync('path/to/certificate.pem')
  };

  https.createServer(options, app).listen(443, () => {
    // HTTPS Server started on port 443
  });
} else {
  // HTTP for development
  app.listen(PORT, () => {
    // HTTP Server started on port ${PORT}
  });
}
```

### Reverse Proxy Configuration

For production, use a reverse proxy like Nginx:

```nginx
server {
    listen 443 ssl http2;
    server_name api.yourdomain.com;

    ssl_certificate /path/to/certificate.pem;
    ssl_certificate_key /path/to/private-key.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Security Checklist

### Development
- [ ] Environment variables configured
- [ ] Stellar keys validated
- [ ] CORS configured for development domains
- [ ] Input validation implemented
- [ ] Error handling secured
- [ ] HTTPS redirect configured

### Production
- [ ] Production Stellar keys configured
- [ ] CORS restricted to production domains
- [ ] Security headers enabled
- [ ] Rate limiting configured
- [ ] HTTPS certificate installed
- [ ] Secrets management implemented
- [ ] Monitoring and alerting configured
- [ ] Regular security audits scheduled

---

*Next: Explore the [API Endpoints](./04-endpoints.md) to understand all available functionality.*