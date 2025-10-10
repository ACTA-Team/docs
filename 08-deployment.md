# Deployment Guide

## Overview

This comprehensive guide covers deploying the ACTA API to various environments, from development to production. It includes best practices for security, scalability, and monitoring.

## **Production Environment Setup**

### **Environment Variables**

Use only the environment variables required by the ACTA API and Soroban contracts. Example production configuration:

```bash
# Core
NODE_ENV=production
PORT=3000

# Stellar network
STELLAR_NETWORK=mainnet
STELLAR_HORIZON_URL=https://horizon.stellar.org
SOROBAN_RPC_URL=https://mainnet.soroban.network

# Issuer key (keep secret)
STELLAR_SECRET_KEY=SBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# ACTA contracts
ACTA_ISSUANCE_CONTRACT_ID=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
ACTA_VAULT_CONTRACT_ID=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
ACTA_DID_CONTRACT_ID=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Optional (only if using the deployer contract)
ACTA_DEPLOYER_CONTRACT_ID=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

### **Container Environment Setup**

Pass the required variables to your container or process manager. Example `docker run`:

```bash
docker run -d \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e PORT=3000 \
  -e STELLAR_NETWORK=mainnet \
  -e STELLAR_HORIZON_URL=https://horizon.stellar.org \
  -e SOROBAN_RPC_URL=https://mainnet.soroban.network \
  -e ACTA_ISSUANCE_CONTRACT_ID=CB... \
  -e ACTA_VAULT_CONTRACT_ID=CB... \
  -e ACTA_DID_CONTRACT_ID=CB... \
  -e ACTA_DEPLOYER_CONTRACT_ID=CB... \
  -e STELLAR_SECRET_KEY=SB... \
  your-account.dkr.ecr.region.amazonaws.com/acta-api:latest
```

---

---

## **Cloud Platform Deployments**

### **AWS Deployment**

#### **Using AWS ECS with Fargate**

```yaml
# aws-ecs-task-definition.json
{
  "family": "acta-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "acta-api",
      "image": "your-account.dkr.ecr.region.amazonaws.com/acta-api:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        { "name": "NODE_ENV", "value": "production" },
        { "name": "STELLAR_NETWORK", "value": "mainnet" },
        { "name": "STELLAR_HORIZON_URL", "value": "https://horizon.stellar.org" },
        { "name": "SOROBAN_RPC_URL", "value": "https://mainnet.soroban.network" },
        { "name": "ACTA_ISSUANCE_CONTRACT_ID", "value": "CB..." },
        { "name": "ACTA_VAULT_CONTRACT_ID", "value": "CB..." },
        { "name": "ACTA_DID_CONTRACT_ID", "value": "CB..." }
      ],
      "secrets": [
        {
          "name": "STELLAR_SECRET_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:acta/stellar-secret-key"
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
        "command": ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

#### **Terraform Configuration for AWS**

```hcl
# main.tf
provider "aws" {
  region = var.aws_region
}

# VPC and Networking
resource "aws_vpc" "acta_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "acta-vpc"
  }
}

resource "aws_subnet" "acta_private_subnet" {
  count             = 2
  vpc_id            = aws_vpc.acta_vpc.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "acta-private-subnet-${count.index + 1}"
  }
}

resource "aws_subnet" "acta_public_subnet" {
  count                   = 2
  vpc_id                  = aws_vpc.acta_vpc.id
  cidr_block              = "10.0.${count.index + 10}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "acta-public-subnet-${count.index + 1}"
  }
}

## Note: No database required
# The ACTA API is stateless and uses Stellar Soroban contracts.
# Remove any RDS/Redis resources; keep networking, ALB, and ECS definitions.

# ECS Cluster
resource "aws_ecs_cluster" "acta_cluster" {
  name = "acta-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

# Application Load Balancer
resource "aws_lb" "acta_alb" {
  name               = "acta-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = aws_subnet.acta_public_subnet[*].id

  enable_deletion_protection = true

  tags = {
    Name = "acta-alb"
  }
}
```

### **Google Cloud Platform (GCP) Deployment**

#### **Using Google Cloud Run**

```yaml
# cloudbuild.yaml
steps:
  # Build the container image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/acta-api:$COMMIT_SHA', '.']
  
  # Push the container image to Container Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/acta-api:$COMMIT_SHA']
  
  # Deploy container image to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
    - 'run'
    - 'deploy'
    - 'acta-api'
    - '--image'
    - 'gcr.io/$PROJECT_ID/acta-api:$COMMIT_SHA'
    - '--region'
    - 'us-central1'
    - '--platform'
    - 'managed'
    - '--allow-unauthenticated'
    - '--set-env-vars'
    - 'NODE_ENV=production,STELLAR_NETWORK=mainnet,STELLAR_HORIZON_URL=https://horizon.stellar.org,SOROBAN_RPC_URL=https://mainnet.soroban.network'
    - '--set-env-vars'
    - 'ACTA_ISSUANCE_CONTRACT_ID=CB...,ACTA_VAULT_CONTRACT_ID=CB...,ACTA_DID_CONTRACT_ID=CB...'
    - '--memory'
    - '2Gi'
    - '--cpu'
    - '2'
    - '--max-instances'
    - '10'
    - '--concurrency'
    - '80'

options:
  logging: CLOUD_LOGGING_ONLY
```

#### **Cloud Run Service Configuration**

```yaml
# service.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: acta-api
  annotations:
    run.googleapis.com/ingress: all
    run.googleapis.com/execution-environment: gen2
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "10"
        autoscaling.knative.dev/minScale: "1"
        run.googleapis.com/cpu-throttling: "false"
        run.googleapis.com/execution-environment: gen2
    spec:
      containerConcurrency: 80
      timeoutSeconds: 300
      containers:
      - image: gcr.io/PROJECT_ID/acta-api:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: STELLAR_NETWORK
          value: "mainnet"
        - name: STELLAR_HORIZON_URL
          value: "https://horizon.stellar.org"
        - name: SOROBAN_RPC_URL
          value: "https://mainnet.soroban.network"
        - name: ACTA_ISSUANCE_CONTRACT_ID
          value: "CB..."
        - name: ACTA_VAULT_CONTRACT_ID
          value: "CB..."
        - name: ACTA_DID_CONTRACT_ID
          value: "CB..."
        - name: STELLAR_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: stellar-secret-key
              key: key
        resources:
          limits:
            cpu: "2"
            memory: "2Gi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### **Microsoft Azure Deployment**

#### **Using Azure Container Instances**

```yaml
# azure-container-group.yaml
apiVersion: 2019-12-01
location: eastus
name: acta-api-group
properties:
  containers:
  - name: acta-api
    properties:
      image: youracr.azurecr.io/acta-api:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 2.0
      ports:
      - port: 3000
        protocol: TCP
      environmentVariables:
      - name: NODE_ENV
        value: production
      - name: STELLAR_NETWORK
        value: mainnet
      - name: STELLAR_HORIZON_URL
        value: https://horizon.stellar.org
      - name: SOROBAN_RPC_URL
        value: https://mainnet.soroban.network
      - name: ACTA_ISSUANCE_CONTRACT_ID
        value: CB...
      - name: ACTA_VAULT_CONTRACT_ID
        value: CB...
      - name: ACTA_DID_CONTRACT_ID
        value: CB...
      secureEnvironmentVariables:
      - name: STELLAR_SECRET_KEY
        secureValue: SB...
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
    - protocol: TCP
      port: 3000
    dnsNameLabel: acta-api
tags:
  environment: production
  project: acta
```

---

## **Kubernetes Deployment**

### **Kubernetes Manifests**

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: acta-production

---
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: acta-config
  namespace: acta-production
  data:
  NODE_ENV: "production"
  STELLAR_NETWORK: "mainnet"
  STELLAR_HORIZON_URL: "https://horizon.stellar.org"
  SOROBAN_RPC_URL: "https://mainnet.soroban.network"
  ACTA_ISSUANCE_CONTRACT_ID: "CB..."
  ACTA_VAULT_CONTRACT_ID: "CB..."
  ACTA_DID_CONTRACT_ID: "CB..."

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: acta-secrets
  namespace: acta-production
type: Opaque
data:
  STELLAR_SECRET_KEY: <base64-encoded-stellar-secret-key>

---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: acta-api
  namespace: acta-production
  labels:
    app: acta-api
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
        envFrom:
        - configMapRef:
            name: acta-config
        - secretRef:
            name: acta-secrets
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: acta-api-service
  namespace: acta-production
spec:
  selector:
    app: acta-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: acta-api-ingress
  namespace: acta-production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
spec:
  tls:
  - hosts:
    - api.yourdomain.com
    secretName: acta-api-tls
  rules:
  - host: api.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: acta-api-service
            port:
              number: 80

---
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: acta-api-hpa
  namespace: acta-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: acta-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

---

## **Monitoring and Observability**

### **Health Check Endpoint**

```javascript
// healthcheck.js
const http = require('http');

const options = {
  hostname: 'localhost',
  port: 3000,
  path: '/health',
  method: 'GET',
  timeout: 3000
};

const req = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

req.on('error', () => {
  process.exit(1);
});

req.on('timeout', () => {
  req.destroy();
  process.exit(1);
});

req.end();
```

### **Prometheus Metrics**

```javascript
// metrics.js
const promClient = require('prom-client');

// Create a Registry
const register = new promClient.Registry();

// Add default metrics
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const credentialOperations = new promClient.Counter({
  name: 'credential_operations_total',
  help: 'Total number of credential operations',
  labelNames: ['operation', 'status']
});

const stellarTransactions = new promClient.Counter({
  name: 'stellar_transactions_total',
  help: 'Total number of Stellar transactions',
  labelNames: ['type', 'status']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(credentialOperations);
register.registerMetric(stellarTransactions);

module.exports = {
  register,
  httpRequestDuration,
  credentialOperations,
  stellarTransactions
};
```

### **Logging Configuration**

```javascript
// logger.js
const winston = require('winston');
const { ElasticsearchTransport } = require('winston-elasticsearch');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'acta-api', version: process.env.npm_package_version },
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error'
    }),
    new winston.transports.File({
      filename: 'logs/combined.log'
    })
  ]
});

// Add Elasticsearch transport for production
if (process.env.NODE_ENV === 'production' && process.env.ELASTICSEARCH_URL) {
  logger.add(new ElasticsearchTransport({
    level: 'info',
    clientOpts: {
      node: process.env.ELASTICSEARCH_URL,
      auth: {
        username: process.env.ELASTICSEARCH_USERNAME,
        password: process.env.ELASTICSEARCH_PASSWORD
      }
    },
    index: 'acta-api-logs'
  }));
}

module.exports = logger;
```

---

## **Security Best Practices**

### **Security Checklist**

- **SSL/TLS Configuration**
  - Use TLS 1.2 or higher
  - Implement proper certificate management
  - Enable HSTS headers

- **API Security**
  - Implement rate limiting
  - Use API keys for authentication
  - Validate all input data
  - Implement CORS properly

- **Infrastructure Security**
  - Use private subnets for databases
  - Implement network security groups
  - Enable encryption at rest and in transit
  - Regular security updates

- **Secrets Management**
  - Use dedicated secret management services
  - Rotate secrets regularly
  - Never commit secrets to version control
  - Use environment-specific secrets

### **Security Headers Middleware**

```javascript
// security.js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const securityMiddleware = (app) => {
  // Helmet for security headers
  app.use(helmet({
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
  }));

  // Rate limiting
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP',
    standardHeaders: true,
    legacyHeaders: false
  });

  app.use('/api/', limiter);

  // Additional security middleware
  app.use((req, res, next) => {
    res.setHeader('X-API-Version', process.env.API_VERSION || 'v1');
    res.setHeader('X-Powered-By', 'ACTA API');
    next();
  });
};

module.exports = securityMiddleware;
```

---

## **Performance Optimization**

### **Caching Strategy**

```javascript
// cache.js
const Redis = require('redis');
const client = Redis.createClient({
  url: process.env.REDIS_URL
});

class CacheManager {
  constructor() {
    this.defaultTTL = 3600; // 1 hour
  }

  async get(key) {
    try {
      const value = await client.get(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }

  async set(key, value, ttl = this.defaultTTL) {
    try {
      await client.setEx(key, ttl, JSON.stringify(value));
    } catch (error) {
      console.error('Cache set error:', error);
    }
  }

  async del(key) {
    try {
      await client.del(key);
    } catch (error) {
      console.error('Cache delete error:', error);
    }
  }

  // Cache middleware for Express
  middleware(ttl = this.defaultTTL) {
    return async (req, res, next) => {
      const key = `cache:${req.method}:${req.originalUrl}`;
      const cached = await this.get(key);

      if (cached) {
        return res.json(cached);
      }

      // Override res.json to cache the response
      const originalJson = res.json;
      res.json = function(data) {
        cache.set(key, data, ttl);
        return originalJson.call(this, data);
      };

      next();
    };
  }
}

const cache = new CacheManager();
module.exports = cache;
```

### **Database Connection Pooling**

```javascript
// database.js
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // maximum number of clients in the pool
  min: 5,  // minimum number of clients in the pool
  idleTimeoutMillis: 30000, // close idle clients after 30 seconds
  connectionTimeoutMillis: 2000, // return an error after 2 seconds if connection could not be established
  maxUses: 7500, // close (and replace) a connection after it has been used 7500 times
});

// Graceful shutdown
process.on('SIGINT', () => {
  pool.end(() => {
    console.log('Database pool has ended');
    process.exit(0);
  });
});

module.exports = pool;
```

---

## **Backup and Disaster Recovery**

### **Database Backup Strategy**

```bash
#!/bin/bash
# backup.sh

# Configuration
DB_HOST="your-db-host"
DB_NAME="acta_prod"
DB_USER="acta"
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/acta_backup_$DATE.sql"

# Create backup directory if it doesn't exist
mkdir -p $BACKUP_DIR

# Create database backup
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME > $BACKUP_FILE

# Compress backup
gzip $BACKUP_FILE

# Upload to cloud storage (AWS S3 example)
aws s3 cp "$BACKUP_FILE.gz" s3://your-backup-bucket/database/

# Clean up old local backups (keep last 7 days)
find $BACKUP_DIR -name "acta_backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_FILE.gz"
```

### **Automated Backup with Cron**

```bash
# Add to crontab: crontab -e
# Run backup daily at 2 AM
0 2 * * * /path/to/backup.sh >> /var/log/acta-backup.log 2>&1

# Run backup every 6 hours
0 */6 * * * /path/to/backup.sh >> /var/log/acta-backup.log 2>&1
```

---

This comprehensive deployment guide ensures your ACTA API is production-ready with proper security, monitoring, and scalability considerations.

*Next: Learn about [Troubleshooting](./09-troubleshooting.md) to resolve common deployment and operational issues.*