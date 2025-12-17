# Getting Started

Quick start guides for different integration scenarios.

## API Integration

Start using the ACTA API to issue and verify credentials:

1. **Choose Network**: Testnet (recommended for development) or Mainnet
2. **Get API Access**: Base URL and network configuration
3. **Issue Credentials**: Use `POST /credentials` endpoint
4. **Verify Credentials**: Use `GET /verify/:vc_id` or `POST /verify`

See [API Developer Quickstart](../api-reference/developer-quickstart.md) for detailed steps.

## React SDK Integration

For React/Next.js applications:

1. **Install SDK**: `npm install @acta/react-sdk`
2. **Configure Provider**: Wrap app with `ActaProvider`
3. **Use Hooks**: `useCreateCredential`, `useVaultApi`, etc.

See [React SDK Documentation](../react-sdk/index.md) for hooks and examples.

## Wallet Integration

Connect Stellar wallets for user authentication and transaction signing:

1. **Install Wallet Kit**: Integrate wallet adapter
2. **Connect Wallet**: User connects Freighter or other Stellar wallet
3. **Sign Transactions**: Use transaction preparation endpoints

See [Wallet Kit Integration](../developer-guide/wallet-kit-quick-integration.md) for details.

## Testnet Setup

Before deploying to mainnet:

1. **Get Testnet Tokens**: Request XLM from Stellar testnet faucet
2. **Configure Network**: Set `NETWORK_TYPE=testnet`
3. **Test Operations**: Issue, store, and verify test credentials
4. **Verify Contracts**: Testnet contract IDs are pre-configured

See [Testnet Tokens](../stellar-network/trustlines/testnet-tokens.md) for faucet links.

## Next Steps

- Review [API Reference](../api-reference/index.md) for all available endpoints
- Check [Schema Documentation](../developer-guide/schema/index.md) for data structures
- Explore [React SDK Hooks](../react-sdk/index.md) for React integration
- Read [Troubleshooting Guide](../contact-and-support/troubleshooting-faqs.md) for common issues

