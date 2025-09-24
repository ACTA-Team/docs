# Deployment and Production Setup

## Overview

This guide covers deploying the ACTA API to production environments, including cloud platforms, containerization, monitoring, and best practices for running a secure and scalable credential management system.

## Pre-Deployment Checklist

### Security Requirements

- [ ] **Environment Variables**: All sensitive data stored in environment variables
- [ ] **Secret Management**: Production secrets managed through secure services
- [ ] **HTTPS**: SSL/TLS certificates configured
- [ ] **CORS**: Proper CORS configuration for production domains
- [ ] **Rate Limiting**: Production-appropriate rate limits configured
- [ ] **Input Validation**: All endpoints properly validate input
- [ ] **Error Handling**: No sensitive information exposed in error messages

### Performance Requirements

- [ ] **Database**: Production database configured and optimized
- [ ] **Caching**: Caching layer implemented where appropriate
- [ ] **Load Balancing**: Load balancer configured for high availability
- [ ] **Monitoring**: Application and infrastructure monitoring set up
- [ ] **Logging**: Centralized logging configured
- [ ] **Backup**: Automated backup strategy implemented

### Stellar Network Requirements

- [ ] **Mainnet Configuration**: Stellar mainnet endpoints configured
- [ ] **Account Funding**: Production Stellar account funded with sufficient XLM
- [ ] **Key Security**: Production Stellar keys securely managed
- [ ] **Contract Deployment**: Smart contracts deployed to mainnet

## Docker Deployment

### Dockerfile

The API includes a production-ready Dockerfile:

```dockerfile
FROM node:18-alpine

# Create app directory
WORKDIR /usr/src/app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Change ownership of the app directory
RUN chown -R nodejs:nodejs /usr/src/app
USER nodejs

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Start the application
CMD ["npm", "start"]
```

### Docker Compose for Production

```yaml
version: '3.8'

services:
  acta-api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - NODE_ENV=production
      - PORT=8000
    env_file:
      - .env.production
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - acta-api
    restart: unless-stopped

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  redis_data:
```

### Building and Running

```bash
# Build the Docker image
docker build -t acta-api:latest .

# Run with Docker Compose
docker-compose -f docker-compose.prod.yml up -d

# Check logs
docker-compose logs -f acta-api

# Scale the application
docker-compose up -d --scale acta-api=3
```

## Cloud Platform Deployments

### AWS Deployment

#### Using AWS ECS (Elastic Container Service)

1. **Create ECS Task Definition**:

```json
{
  "family": "acta-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::ACCOUNT:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::ACCOUNT:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "acta-api",
      "image": "your-account.dkr.ecr.region.amazonaws.com/acta-api:latest",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        },
        {
          "name": "PORT",
          "value": "8000"
        }
      ],
      "secrets": [
        {
          "name": "STELLAR_SECRET_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:stellar-secret-key"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/acta-api",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

2. **Create ECS Service**:

```bash
aws ecs create-service \
  --cluster acta-cluster \
  --service-name acta-api-service \
  --task-definition acta-api:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345,subnet-67890],securityGroups=[sg-abcdef],assignPublicIp=ENABLED}" \
  --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/acta-api-tg,containerName=acta-api,containerPort=8000
```

#### Using AWS Lambda (Serverless)

1. **Install Serverless Framework**:

```bash
npm install -g serverless
npm install serverless-http
```

2. **Create serverless.yml**:

```yaml
service: acta-api

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
  environment:
    NODE_ENV: production
    STELLAR_HORIZON_URL: https://horizon.stellar.org
    STELLAR_SOROBAN_URL: https://soroban.stellar.org
    STELLAR_NETWORK_PASSPHRASE: Public Global Stellar Network ; September 2015
  iamRoleStatements:
    - Effect: Allow
      Action:
        - secretsmanager:GetSecretValue
      Resource: 
        - arn:aws:secretsmanager:${self:provider.region}:*:secret:stellar-secret-key*

functions:
  api:
    handler: lambda.handler
    events:
      - http:
          path: /{proxy+}
          method: ANY
          cors: true
      - http:
          path: /
          method: ANY
          cors: true
    timeout: 30
    memorySize: 512

plugins:
  - serverless-offline
```

3. **Create Lambda Handler**:

```javascript
// lambda.js
const serverless = require('serverless-http');
const app = require('./src/index');

module.exports.handler = serverless(app);
```

### Google Cloud Platform (GCP)

#### Using Cloud Run

1. **Create Dockerfile** (same as above)

2. **Deploy to Cloud Run**:

```bash
# Build and push to Container Registry
gcloud builds submit --tag gcr.io/PROJECT_ID/acta-api

# Deploy to Cloud Run
gcloud run deploy acta-api \
  --image gcr.io/PROJECT_ID/acta-api \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars NODE_ENV=production \
  --set-secrets STELLAR_SECRET_KEY=stellar-secret-key:latest \
  --port 8000 \
  --memory 1Gi \
  --cpu 1 \
  --min-instances 1 \
  --max-instances 10
```

#### Using Google Kubernetes Engine (GKE)

1. **Create Kubernetes Deployment**:

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
        image: gcr.io/PROJECT_ID/acta-api:latest
        ports:
        - containerPort: 8000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "8000"
        - name: STELLAR_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: stellar-secrets
              key: secret-key
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: acta-api-service
spec:
  selector:
    app: acta-api
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer
```

### Microsoft Azure

#### Using Azure Container Instances

```bash
# Create resource group
az group create --name acta-api-rg --location eastus

# Create container instance
az container create \
  --resource-group acta-api-rg \
  --name acta-api \
  --image your-registry/acta-api:latest \
  --dns-name-label acta-api-unique \
  --ports 8000 \
  --environment-variables NODE_ENV=production PORT=8000 \
  --secure-environment-variables STELLAR_SECRET_KEY=your-secret-key \
  --cpu 1 \
  --memory 2
```

#### Using Azure App Service

```bash
# Create App Service plan
az appservice plan create \
  --name acta-api-plan \
  --resource-group acta-api-rg \
  --sku B1 \
  --is-linux

# Create web app
az webapp create \
  --resource-group acta-api-rg \
  --plan acta-api-plan \
  --name acta-api-webapp \
  --deployment-container-image-name your-registry/acta-api:latest

# Configure app settings
az webapp config appsettings set \
  --resource-group acta-api-rg \
  --name acta-api-webapp \
  --settings NODE_ENV=production PORT=8000 STELLAR_SECRET_KEY=your-secret-key
```

## Load Balancing and High Availability

### Nginx Configuration

```nginx
upstream acta_api {
    least_conn;
    server acta-api-1:8000 max_fails=3 fail_timeout=30s;
    server acta-api-2:8000 max_fails=3 fail_timeout=30s;
    server acta-api-3:8000 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.yourdomain.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req zone=api burst=20 nodelay;

    location / {
        proxy_pass http://acta_api;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    location /health {
        proxy_pass http://acta_api;
        access_log off;
    }
}
```

### HAProxy Configuration

```
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    option httplog

frontend acta_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/acta-api.pem
    redirect scheme https if !{ ssl_fc }
    
    # Rate limiting
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request reject if { sc_http_req_rate(0) gt 20 }
    
    default_backend acta_backend

backend acta_backend
    balance roundrobin
    option httpchk GET /health
    server api1 acta-api-1:8000 check
    server api2 acta-api-2:8000 check
    server api3 acta-api-3:8000 check
```

## Monitoring and Observability

### Application Monitoring

#### Prometheus Metrics

```javascript
// src/middleware/metrics.js
const promClient = require('prom-client');

// Create metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const stellarOperations = new promClient.Counter({
  name: 'stellar_operations_total',
  help: 'Total number of Stellar operations',
  labelNames: ['operation_type', 'status']
});

// Middleware
const metricsMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route?.path || req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);
    
    httpRequestTotal
      .labels(req.method, route, res.statusCode)
      .inc();
  });
  
  next();
};

module.exports = {
  metricsMiddleware,
  stellarOperations,
  register: promClient.register
};
```

#### Health Check Endpoint

```javascript
// Enhanced health check
app.get('/health', async (req, res) => {
  const health = {
    status: 'OK',
    timestamp: new Date().toISOString(),
    service: 'ACTA API',
    version: process.env.npm_package_version,
    checks: {}
  };

  try {
    // Check Stellar connection
    const stellarHealth = await checkStellarConnection();
    health.checks.stellar = stellarHealth;

    // Check database connection (if applicable)
    // const dbHealth = await checkDatabaseConnection();
    // health.checks.database = dbHealth;

    // Check memory usage
    const memUsage = process.memoryUsage();
    health.checks.memory = {
      status: memUsage.heapUsed < 500 * 1024 * 1024 ? 'OK' : 'WARNING',
      heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)}MB`,
      heapTotal: `${Math.round(memUsage.heapTotal / 1024 / 1024)}MB`
    };

    const allHealthy = Object.values(health.checks).every(check => check.status === 'OK');
    
    if (!allHealthy) {
      health.status = 'DEGRADED';
      return res.status(503).json(health);
    }

    res.json(health);
  } catch (error) {
    health.status = 'ERROR';
    health.error = error.message;
    res.status(503).json(health);
  }
});

async function checkStellarConnection() {
  try {
    const server = new Server(process.env.STELLAR_HORIZON_URL);
    await server.ledgers().limit(1).call();
    return { status: 'OK', message: 'Stellar connection healthy' };
  } catch (error) {
    return { status: 'ERROR', message: error.message };
  }
}
```

### Logging

#### Structured Logging with Winston

```javascript
// src/config/logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'acta-api' },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

module.exports = logger;
```

#### Request Logging Middleware

```javascript
// src/middleware/logging.js
const logger = require('../config/logger');

const requestLogger = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('User-Agent'),
      ip: req.ip
    });
  });
  
  next();
};

module.exports = requestLogger;
```

### Error Tracking

#### Sentry Integration

```javascript
// src/config/sentry.js
const Sentry = require('@sentry/node');

if (process.env.NODE_ENV === 'production') {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 0.1
  });
}

module.exports = Sentry;
```

## Security Hardening

### Environment Security

```bash
# .env.production
NODE_ENV=production
PORT=8000

# Stellar Configuration
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_HORIZON_URL=https://horizon.stellar.org
STELLAR_SOROBAN_URL=https://soroban.stellar.org
STELLAR_NETWORK_PASSPHRASE=Public Global Stellar Network ; September 2015

# Security
ALLOWED_ORIGINS=https://app.yourdomain.com,https://admin.yourdomain.com
SESSION_SECRET=your-super-secret-session-key
JWT_SECRET=your-jwt-secret-key

# Monitoring
SENTRY_DSN=https://your-sentry-dsn
LOG_LEVEL=warn

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

### Security Middleware

```javascript
// src/middleware/security.js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

// Rate limiting
const createRateLimit = (windowMs, max, message) => rateLimit({
  windowMs,
  max,
  message: { error: message },
  standardHeaders: true,
  legacyHeaders: false
});

const generalLimiter = createRateLimit(
  15 * 60 * 1000, // 15 minutes
  100, // limit each IP to 100 requests per windowMs
  'Too many requests from this IP'
);

const credentialLimiter = createRateLimit(
  60 * 1000, // 1 minute
  5, // limit each IP to 5 credential creations per minute
  'Too many credential creation attempts'
);

// Security headers
const securityHeaders = helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
});

module.exports = {
  generalLimiter,
  credentialLimiter,
  securityHeaders
};
```

## Backup and Disaster Recovery

### Database Backup Strategy

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DB_NAME="acta_api"

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup (if using PostgreSQL)
pg_dump $DB_NAME > $BACKUP_DIR/db_backup_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/db_backup_$DATE.sql

# Upload to cloud storage (AWS S3 example)
aws s3 cp $BACKUP_DIR/db_backup_$DATE.sql.gz s3://acta-backups/database/

# Clean up old local backups (keep last 7 days)
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: db_backup_$DATE.sql.gz"
```

### Stellar Account Backup

```javascript
// scripts/backup-stellar-account.js
const { Keypair, Server } = require('stellar-sdk');

async function backupStellarAccount() {
  const keypair = Keypair.fromSecret(process.env.STELLAR_SECRET_KEY);
  const server = new Server(process.env.STELLAR_HORIZON_URL);
  
  try {
    const account = await server.loadAccount(keypair.publicKey());
    
    const backup = {
      publicKey: keypair.publicKey(),
      accountId: account.accountId(),
      sequence: account.sequence,
      balances: account.balances,
      signers: account.signers,
      data: account.data_attr,
      flags: account.flags,
      thresholds: account.thresholds,
      backupDate: new Date().toISOString()
    };
    
    // Backup created successfully - save to secure storage
    // await saveToSecureStorage(backup);
    return backup;
    
  } catch (error) {
    // Backup failed - handle error appropriately
    throw new Error(`Backup failed: ${error.message}`);
  }
}

if (require.main === module) {
  backupStellarAccount();
}
```

## Performance Optimization

### Caching Strategy

```javascript
// src/middleware/cache.js
const redis = require('redis');
const client = redis.createClient(process.env.REDIS_URL);

const cache = (duration = 300) => {
  return async (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    
    try {
      const cached = await client.get(key);
      if (cached) {
        return res.json(JSON.parse(cached));
      }
      
      // Store original json method
      const originalJson = res.json;
      
      // Override json method to cache response
      res.json = function(data) {
        client.setex(key, duration, JSON.stringify(data));
        originalJson.call(this, data);
      };
      
      next();
    } catch (error) {
      // Cache error - continue without caching
      next();
    }
  };
};

module.exports = cache;
```

### Connection Pooling

```javascript
// src/config/stellar.js
const { Server, SorobanRpc } = require('stellar-sdk');

class StellarConnectionPool {
  constructor() {
    this.horizonServers = [
      new Server(process.env.STELLAR_HORIZON_URL),
      new Server(process.env.STELLAR_HORIZON_URL_BACKUP)
    ].filter(Boolean);
    
    this.sorobanServers = [
      new SorobanRpc.Server(process.env.STELLAR_SOROBAN_URL),
      new SorobanRpc.Server(process.env.STELLAR_SOROBAN_URL_BACKUP)
    ].filter(Boolean);
    
    this.currentHorizonIndex = 0;
    this.currentSorobanIndex = 0;
  }
  
  getHorizonServer() {
    const server = this.horizonServers[this.currentHorizonIndex];
    this.currentHorizonIndex = (this.currentHorizonIndex + 1) % this.horizonServers.length;
    return server;
  }
  
  getSorobanServer() {
    const server = this.sorobanServers[this.currentSorobanIndex];
    this.currentSorobanIndex = (this.currentSorobanIndex + 1) % this.sorobanServers.length;
    return server;
  }
}

module.exports = new StellarConnectionPool();
```

## Maintenance and Updates

### Zero-Downtime Deployment

```bash
#!/bin/bash
# deploy.sh

set -e

echo "Starting zero-downtime deployment..."

# Build new image
docker build -t acta-api:$BUILD_NUMBER .

# Tag as latest
docker tag acta-api:$BUILD_NUMBER acta-api:latest

# Update service (rolling update)
docker service update --image acta-api:latest acta-api-service

# Wait for deployment to complete
echo "Waiting for deployment to complete..."
sleep 30

# Health check
if curl -f http://localhost:8000/health; then
  echo "✅ Deployment successful"
else
  echo "❌ Deployment failed, rolling back..."
  docker service rollback acta-api-service
  exit 1
fi

# Clean up old images
docker image prune -f

echo "Deployment completed successfully"
```

### Database Migrations

```javascript
// migrations/001_initial_schema.js
exports.up = function(knex) {
  return knex.schema.createTable('credentials', function(table) {
    table.string('contract_id').primary();
    table.string('hash').notNullable().unique();
    table.string('status').notNullable().defaultTo('Active');
    table.json('metadata');
    table.timestamps(true, true);
    
    table.index('hash');
    table.index('status');
  });
};

exports.down = function(knex) {
  return knex.schema.dropTable('credentials');
};
```

This comprehensive deployment guide ensures your ACTA API runs securely and efficiently in production environments with proper monitoring, backup strategies, and maintenance procedures.

---

*Next: Learn about [Troubleshooting and FAQ](./09-troubleshooting.md) to resolve common issues and questions.*