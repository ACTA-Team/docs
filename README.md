# ACTA API Documentation

**ACTA API**
*Autonomous Credential Trust Architecture*

---

## Table of Contents

This documentation is organized into the following sections:

| Section | Description | Status |
|---------|-------------|--------|
| **[Introduction](./01-introduction.md)** | Overview of the ACTA API, architecture, and core concepts | Complete |
| **[Setup and Installation](./02-setup.md)** | Complete setup guide for development and testing | Complete |
| **[Authentication and Security](./03-authentication.md)** | Security implementation and best practices | Complete |
| **[API Endpoints](./04-endpoints.md)** | Detailed reference for all API endpoints | Complete |
| **[Services](./05-services.md)** | Core services and business logic documentation | Complete |
| **[Configuration](./06-configuration.md)** | Environment variables and configuration options | Complete |
| **[Examples and Use Cases](./07-examples.md)** | Practical examples and integration patterns | Complete |
| **[Deployment Guide](./08-deployment.md)** | Production deployment and infrastructure setup | Complete |
| **[Troubleshooting and FAQ](./09-troubleshooting.md)** | Common issues and solutions | Complete |

## Quick Start

### Get up and running in 3 simple steps!

### 1️⃣ **Clone and Setup**
```bash
git clone <repository-url>
cd acta-api
cp .env.example .env
```

### 2️⃣ **Configure Environment**
Edit `.env` with your Stellar network configuration:
```bash
# Required: Your Stellar secret key
STELLAR_SECRET_KEY=your_stellar_secret_key_here

# Network configuration (testnet/mainnet)
STELLAR_NETWORK=testnet
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
```

### 3️⃣ **Install and Run**
```bash
npm install
npm run dev
```

### **Test the API**
```bash
# Health check
curl https://acta.up.railway.app/health

# Or locally
curl http://localhost:8000/health
```

**Congratulations!** Your ACTA API is now running!

## Architecture Overview

The ACTA API follows a modular architecture designed for scalability and maintainability:

### **Core Components**

| Component | Technology | Purpose |
|-----------|------------|----------|
| **Express.js Server** | Node.js + TypeScript | High-performance web server with type safety |
| **Stellar SDK Integration** | @stellar/stellar-sdk | Direct blockchain interaction and transaction handling |
| **Soroban Smart Contracts** | Rust + Soroban | On-chain business logic execution |
| **Security Layer** | Helmet + CORS | Comprehensive security headers and access control |
| **Database Layer** | Supabase PostgreSQL | Persistent data storage with encryption |

## Getting Help

**Documentation**: Check our comprehensive documentation
**Community**: Join our community discussions  
**Issues**: Report bugs and request features
**Support**: Get help from our development team

## Useful Links

**Stellar Documentation**: Learn about Stellar blockchain
**Soroban Smart Contracts**: Explore Soroban development
**GitHub Repository**: Access the source code
**Complete Documentation**: Full documentation on GitBook

---

---

*This documentation is maintained by the ACTA development team and is regularly updated to reflect the latest API changes and best practices.*

**Version 1.0.0** | **Last Updated: January 2025** | **Status: Active**

**Current Version**: 1.0.0  
**Last Updated**: January 2024  
**API Compatibility**: Stellar Mainnet & Testnet

---

*This documentation is designed for developers who want to integrate or contribute to the ACTA ecosystem.*