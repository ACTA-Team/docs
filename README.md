# ACTA API Documentation

## Complete Documentation for Stellar Verifiable Credentials Management API

Welcome to the official documentation of **ACTA API v1.0.0**, a robust and secure REST API for creating, managing, and verifying verifiable credentials using Stellar blockchain and Soroban smart contracts.

## 🌟 Key Features

- **Verifiable Credentials Management**: Creation, querying, and updating of blockchain-based credentials
- **Stellar Integration**: Uses Stellar network for immutable storage
- **Soroban Smart Contracts**: Business logic implementation on blockchain
- **Advanced Security**: Security headers, configured CORS, and robust validations
- **RESTful API**: Well-structured endpoints following REST standards
- **Multi-network Support**: Compatible with Stellar Testnet and Mainnet

## 📚 Table of Contents

1. [**Introduction**](./01-introduction.md)
   - What is ACTA API
   - System architecture
   - Use cases

2. [**Setup and Installation**](./02-setup.md)
   - System requirements
   - Environment variables
   - Step-by-step installation
   - Stellar configuration

3. [**Authentication and Security**](./03-authentication.md)
   - CORS configuration
   - Security headers
   - Stellar key validation

4. [**API Endpoints**](./04-endpoints.md)
   - Complete documentation of all endpoints
   - Request/response examples
   - HTTP status codes

5. [**Services and Business Logic**](./05-services.md)
   - StellarService
   - Smart contract management
   - Validations and utilities

6. [**Types and Data Structures**](./06-types.md)
   - TypeScript interfaces
   - Enums and constants
   - Data models

7. [**Practical Examples**](./07-examples.md)
   - Complete use cases
   - Example code
   - Frontend integration

8. [**Error Handling**](./08-errors.md)
   - Error codes
   - Error messages
   - Debugging and troubleshooting

9. [**Deployment and Production**](./09-deployment.md)
   - Production configuration
   - Docker and containers
   - Monitoring and logs

10. [**Technical Reference**](./10-reference.md)
    - Technical specifications
    - Dependencies
    - Contract architecture

## 🚀 Quick Start

```bash
# Clone the repository
git clone <repository-url>

# Navigate to directory
cd ACTA-api

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env

# Run in development mode
npm run dev
```

The API will be available at `http://localhost:3000`

## 🔗 Useful Links

- [Stellar Developer Portal](https://developers.stellar.org/)
- [Soroban Documentation](https://soroban.stellar.org/)
- [Stellar Laboratory](https://laboratory.stellar.org/)

## 📝 Version

**Current version**: 1.0.0  
**Last updated**: January 2025

---

*This documentation is designed for developers who want to integrate or contribute to the ACTA ecosystem.*