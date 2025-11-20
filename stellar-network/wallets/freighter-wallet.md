# Freighter Wallet

Browser extension wallet for Stellar. Non-custodial and suitable for testnet and mainnet.

## What You’ll Learn

- Install and set up Freighter
- Connect Freighter to your ACTA-based application
- Tips and resources

## Installation

- Open the official Freighter website
- Click “Add to Browser” for Chrome/Brave/Firefox
- Install only from official sources
- Pin the extension for easy access

## Setup

### Create a New Wallet

- Open the extension and choose “Create New Wallet”
- Set a secure password
- Write down the recovery phrase (seed) and store it safely

### Import an Existing Wallet

- Open the extension and choose “Import Wallet”
- Enter your seed phrase and set a password

## Connect Freighter to Your App

- Navigate to your application
- Click “Connect Wallet” and select “Freighter”
- Approve the connection in the extension
- Ensure network is set correctly in Freighter settings (Testnet or Mainnet)

## Use with ACTA

- Integrate signing via the Wallet Kit guide: `docs/developer-guide/wallet-kit-quick-integration.md`
- Pass a signer with signature `(unsignedXdr: string, opts: { networkPassphrase: string }) => Promise<string>` to ACTA hooks
- Obtain `networkPassphrase` using `client.getDefaults()` (`sdk/src/client.ts:55-63`)

## Best Practices

- Backup the seed phrase offline
- Use Testnet for development to avoid real fund loss
- Keep browser secure and avoid unknown extensions

## Resources

- Official website: Freighter Wallet
- Documentation: Freighter GitHub repository