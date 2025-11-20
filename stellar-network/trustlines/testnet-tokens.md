# Testnet Tokens

Virtual assets on Stellar testnet for learning and development.

## What Are Testnet Tokens?

- Used to test wallet setups, payment flows, and contract interactions
- Ideal for practicing trustlines and submitting pre-signed transactions
- Have no monetary value and are reset periodically with the testnet

## Why Use Testnet Tokens?

- Risk-free learning and rapid prototyping
- Validate dApp integrations and transaction signing before mainnet
- Community-wide testing without real funds

## Get XLM on Testnet

- Use Stellar Laboratory faucet
  - Open the Stellar Laboratory
  - Paste your public key
  - Click “Request Lumens” and confirm the transaction
  - Check your wallet; you should receive test XLM

## Get USDC on Testnet

- Set a trustline to the USDC issuer for the selected network
  - Mainnet issuer: `CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75`
  - Testnet issuer: `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA`
  - Source: `sdk/src/types/types.ts:17-24`
- Request testnet USDC from available ecosystem faucets or test distributions

## How to Use Testnet Tokens

- Explore wallet operations: send tokens between test accounts, test memos and fees
- Learn key concepts: create trustlines, issue custom assets, simulate contract-like flows
- Connect to applications: use your testnet wallet with ACTA dApps and hooks

## Notes

- Tokens have no value and cannot be moved to mainnet
- Testnet resets periodically; re-create and re-fund accounts after resets
- Never share secret keys; follow mainnet-grade security practices

## Troubleshooting

- If tokens don’t arrive: verify the public key, retry the faucet, and wait briefly
- If wallet cannot switch networks: ensure testnet support (e.g., Freighter, Albedo)
- Lost secret key: access to the account is not recoverable