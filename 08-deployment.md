# Deployment

<div align="center">

![Deployment](https://img.shields.io/badge/Deployment-Production%20Ready-blue?style=for-the-badge&logo=rocket&logoColor=white)

</div>

## Overview

This guide covers deploying the ACTA API to production environments, including cloud platforms, containerization, monitoring, and best practices for scalable and secure deployments.

---

## **Production Environment Setup**

### **Environment Variables**

Configure these essential environment variables for production:

```bash
# Core Configuration
NODE_ENV=production
PORT=3000
API_VERSION=v1

# Stellar Network Configuration
STELLAR_NETWORK=mainnet
STELLAR_HORIZON_URL=https://horizon.stellar.org
STELLAR_PASSPHRASE="Public Global Stellar Network ; September 2015"

# Database Configuration
DATABASE_URL=postgresql://user:password@host:port/database
REDIS_URL=redis://user:password@host:port

# Security
JWT_SECRET=your-super-secure-jwt-secret-key
API_KEY_SECRET=your-api-key-encryption-secret
CORS_ORIGIN=https://yourdomain.com

# Monitoring & Logging
LOG_LEVEL=info
SENTRY_DSN=https://your-sentry-dsn
DATADOG_API_KEY=your-datadog-api-key

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# SSL/TLS
SSL_CERT_PATH=/path/to/ssl/cert.pem
SSL_KEY_PATH=/path/to/ssl/private.key
```

### **Production Configuration File**

Create a production configuration file:

```javascript
// config/production.js
module.exports = {
  server: {
    port: process.env.PORT || 3000,
    host: '0.0.0.0',
    ssl: {
      enabled: true,
      cert: process.env.SSL_CERT_PATH,
      key: process.env.SSL_KEY_PATH
    }
  },
  
  stellar: {
    network: 'mainnet',
    horizonUrl: 'https://horizon.stellar.org',
    passphrase: 'Public Global Stellar Network ; September 2015'
  },
  
  database: {
    url: process.env.DATABASE_URL,
    pool: {
      min: 5,
      max: 20,
      idle: 10000,
      acquire: 30000
    },
    logging: false
  },
  
  redis: {
    url: process.env.REDIS_URL,
    retryDelayOnFailover: 100,
    maxRetriesPerRequest: 3
  },
  
  security: {
    jwtSecret: process.env.JWT_SECRET,
    apiKeySecret: process.env.API_KEY_SECRET,
    cors: {
      origin: process.env.CORS_ORIGIN,
      credentials: true
    },
    helmet: {
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          styleSrc: ["'self'", "'unsafe-inline'"],
          scriptSrc: ["'self'"],
          imgSrc: ["'self'", "data:", "https:"]
        }
      }
    }
  },
  
  rateLimit: {
    windowMs: parseInt(process.env.RATE_LIMIT_WINDOW_MS) || 900000,
    max: parseInt(process.env.RATE_LIMIT_MAX_REQUESTS) || 100,
    standardHeaders: true,
    legacyHeaders: false
  },
  
  logging: {
    level: process.env.LOG_LEVEL || 'info',
    format: 'json',
    transports: ['console', 'file']
  }
};
```

---

## **Docker Deployment**

### **Dockerfile**

Create an optimized production Dockerfile:

```dockerfile
# Multi-stage build for production
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy source code
COPY . .

# Build the application (if you have a build step)
RUN npm run build

# Production stage
FROM node:18-alpine AS production

# Create app user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S acta -u 1001

# Set working directory
WORKDIR /app

# Copy built application from builder stage
COPY --from=builder --chown=acta:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=acta:nodejs /app/dist ./dist
COPY --from=builder --chown=acta:nodejs /app/package*.json ./

# Install security updates
RUN apk update && apk upgrade && apk add --no-cache dumb-init

# Switch to non-root user
USER acta

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]

# Start the application
CMD ["node", "dist/server.js"]
```

### **Docker Compose for Production**

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  acta-api:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://acta:${DB_PASSWORD}@postgres:5432/acta_prod
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    networks:
      - acta-network
    volumes:
      - ./logs:/app/logs
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=acta_prod
      - POSTGRES_USER=acta
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    restart: unless-stopped
    networks:
      - acta-network
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - acta-network

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
    networks:
      - acta-network

volumes:
  postgres_data:
  redis_data:

networks:
  acta-network:
    driver: bridge
```

### **Nginx Configuration**

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream acta_api {
        server acta-api:3000;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    server {
        listen 80;
        server_name yourdomain.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name yourdomain.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/private.key;

        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # API routes
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            
            proxy_pass http://acta_api;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 30s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
        }

        # Health check
        location /health {
            proxy_pass http://acta_api/health;
            access_log off;
        }
    }
}
```

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
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:acta/database-url"
        },
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:acta/jwt-secret"
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

# RDS Database
resource "aws_db_instance" "acta_db" {
  identifier     = "acta-database"
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = "db.t3.micro"
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_type          = "gp2"
  storage_encrypted     = true

  db_name  = "acta_prod"
  username = "acta"
  password = var.db_password

  vpc_security_group_ids = [aws_security_group.rds_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.acta_db_subnet_group.name

  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"

  skip_final_snapshot = false
  final_snapshot_identifier = "acta-db-final-snapshot"

  tags = {
    Name = "acta-database"
  }
}

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
    - 'NODE_ENV=production'
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
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-url
              key: url
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
      - name: DATABASE_URL
        secureValue: postgresql://user:pass@host:5432/db
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
  LOG_LEVEL: "info"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: acta-secrets
  namespace: acta-production
type: Opaque
data:
  DATABASE_URL: <base64-encoded-database-url>
  JWT_SECRET: <base64-encoded-jwt-secret>
  API_KEY_SECRET: <base64-encoded-api-key-secret>

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