# Setup and Installation

## System Requirements

Before installing the ACTA API, ensure your system meets the following requirements:

### Minimum Requirements
- **Node.js**: Version 18.0 or higher
- **npm**: Version 8.0 or higher (comes with Node.js)
- **Memory**: 2GB RAM minimum
- **Storage**: 1GB free disk space
- **Network**: Stable internet connection for Stellar network access

### Recommended Requirements
- **Node.js**: Version 20.0 or higher
- **npm**: Version 10.0 or higher
- **Memory**: 4GB RAM or more
- **Storage**: 5GB free disk space
- **Network**: High-speed internet connection

### Operating System Support
- **Linux**: Ubuntu 20.04+, CentOS 8+, Debian 11+
- **macOS**: macOS 12.0 (Monterey) or higher
- **Windows**: Windows 10/11 with WSL2 recommended

## Installation Steps

### 1. Clone the Repository

```bash
# Clone the ACTA API repository
git clone https://github.com/your-org/ACTA-api.git

# Navigate to the project directory
cd ACTA-api
```

### 2. Install Dependencies

```bash
# Install all required dependencies
npm install

# Verify installation
npm list --depth=0
```

### 3. Environment Configuration

Copy the example environment file and configure it:

```bash
# Copy the example environment file
cp .env.example .env
```

Edit the `.env` file with your configuration:

```env
# Stellar Configuration
# ===================

# Stellar Network: "mainnet" or "testnet"
STELLAR_NETWORK=testnet

# Stellar Secret Key (required)
# For testnet, you can get one from: https://laboratory.stellar.org/#account-creator
# For mainnet, use your actual funded account secret key
# IMPORTANT: Keep this secret and never commit it to version control!
STELLAR_SECRET_KEY=SXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Stellar Horizon Server URL (optional)
# Leave empty to use default for the selected network
# Testnet default: https://horizon-testnet.stellar.org
# Mainnet default: https://horizon.stellar.org
STELLAR_HORIZON_URL=

# Server Configuration
# ===================

# Port for the API server
PORT=3000

# Node environment
NODE_ENV=development

# Security Configuration
# =====================

# JWT Secret for authentication (if implemented)
JWT_SECRET=your-super-secret-jwt-key-here

# API Key Secret for API key generation (if implemented)
API_KEY_SECRET=your-api-key-secret-here

# CORS Configuration
# ==================

# Allowed origins for CORS (comma-separated)
ALLOWED_ORIGINS=http://localhost:8000,http://localhost:3001,https://yourdomain.com

# Database Configuration (if applicable)
# ====================================

# Database connection string (if using a database)
DATABASE_URL=postgresql://username:password@localhost:5432/acta_db

# Logging Configuration
# ====================

# Log level: error, warn, info, debug
LOG_LEVEL=info

# Log format: json, simple
LOG_FORMAT=simple
```

## Stellar Network Setup

### Getting a Stellar Account

#### For Testnet (Development)

1. Visit the [Stellar Laboratory](https://laboratory.stellar.org/#account-creator)
2. Click "Generate keypair" to create a new account
3. Copy the **Secret Key** (starts with 'S')
4. Click "Fund account" to add test XLM to your account
5. Add the secret key to your `.env` file

#### For Mainnet (Production)

1. Create a Stellar account using a wallet like:
   - [Stellar Wallet](https://wallet.stellar.org/)
   - [Lobstr](https://lobstr.co/)
   - [StellarTerm](https://stellarterm.com/)
2. Fund your account with XLM (minimum 5 XLM recommended)
3. Export your secret key securely
4. Add the secret key to your `.env` file

### Verifying Stellar Configuration

Run the verification script to ensure your Stellar setup is correct:

```bash
# Verify Stellar configuration
npm run verify-stellar
```

If the verification script doesn't exist, you can manually verify by starting the API:

```bash
# Start the API in development mode
npm run dev
```

Look for these log messages:
```
✅ Using Stellar account: GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
🌐 Stellar Network: Testnet
🔗 Horizon URL: https://horizon-testnet.stellar.org
```

## Development Setup

### 1. Start Development Server

```bash
# Start the API in development mode with hot reload
npm run dev
```

The API will be available at `http://localhost:8000` (or your configured PORT).

### 2. Verify Installation

Test the API endpoints:

```bash
# Test the root endpoint
curl http://localhost:8000/

# Test the health endpoint
curl http://localhost:8000/health
```

Expected responses:

**Root endpoint (`/`)**:
```json
{
  "message": "ACTA API - Stellar Credential Management",
  "version": "1.0.0",
  "endpoints": {
    "health": "/health",
    "credentials": "/credentials"
  }
}
```

**Health endpoint (`/health`)**:
```json
{
  "status": "OK",
  "timestamp": "2025-01-XX:XX:XX.XXXZ",
  "service": "Stellar Credential API"
}
```

## Production Setup

### 1. Build the Application

```bash
# Build the TypeScript code
npm run build

# Start the production server
npm start
```

### 2. Environment Variables for Production

Update your `.env` file for production:

```env
# Production configuration
NODE_ENV=production
STELLAR_NETWORK=mainnet
PORT=8000

# Use production Stellar account
STELLAR_SECRET_KEY=your-production-secret-key

# Production security settings
JWT_SECRET=your-production-jwt-secret
API_KEY_SECRET=your-production-api-key-secret

# Production CORS settings
ALLOWED_ORIGINS=https://yourdomain.com,https://app.yourdomain.com

# Production logging
LOG_LEVEL=warn
LOG_FORMAT=json
```

### 3. Process Management

For production deployment, use a process manager like PM2:

```bash
# Install PM2 globally
npm install -g pm2

# Start the application with PM2
pm2 start dist/index.js --name "acta-api"

# Save PM2 configuration
pm2 save

# Setup PM2 to start on system boot
pm2 startup
```

## Docker Setup

### 1. Using Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  acta-api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - STELLAR_NETWORK=testnet
      - STELLAR_SECRET_KEY=${STELLAR_SECRET_KEY}
      - PORT=3000
    volumes:
      - ./contracts:/app/contracts
    restart: unless-stopped

  # Optional: Add a reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - acta-api
    restart: unless-stopped
```

### 2. Build and Run

```bash
# Build and start the containers
docker-compose up -d

# View logs
docker-compose logs -f acta-api

# Stop the containers
docker-compose down
```

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