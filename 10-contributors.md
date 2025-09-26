# Contributors Guide

## Development Setup

This guide is for developers who want to contribute to ACTA or run their own instance of the API.

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

### 4. Start Development Server

```bash
# Start the development server
npm run dev

# Or start in production mode
npm start
```

## Development Workflow

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

### Code Quality

```bash
# Run linting
npm run lint

# Fix linting issues
npm run lint:fix

# Format code
npm run format
```

### Building for Production

```bash
# Build the project
npm run build

# Start production server
npm run start:prod
```

## Contributing Guidelines

### Code Style
- Use TypeScript for all new code
- Follow the existing code style and conventions
- Write tests for new features
- Update documentation when needed

### Pull Request Process
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Write/update tests
5. Update documentation
6. Submit a pull request

### Commit Messages
Use conventional commit format:
```
feat: add new credential validation
fix: resolve stellar network timeout
docs: update API documentation
test: add unit tests for credential service
```

## Stellar Network Setup for Development

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

## Troubleshooting Development Issues

### Common Issues

1. **Node.js Version Mismatch**
   ```bash
   # Use nvm to manage Node.js versions
   nvm use 20
   ```

2. **Port Already in Use**
   ```bash
   # Change port in .env file
   PORT=3001
   ```

3. **Stellar Network Connection Issues**
   - Check your internet connection
   - Verify STELLAR_HORIZON_URL in .env
   - Ensure your Stellar account is funded

### Getting Help

- Check the [Troubleshooting Guide](./09-troubleshooting.md)
- Open an issue on GitHub
- Join our Discord community