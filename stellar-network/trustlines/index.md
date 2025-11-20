# Trustlines

On Stellar, accounts must explicitly opt in to hold and use assets. This opt‑in is a trustline.

## What Are Trustlines?

- A trustline authorizes an account to hold, receive, and transact with a non‑native asset (anything other than XLM) from a specific issuer.
- Without a trustline, an account cannot receive or keep that asset.
- Each trustline requires ~0.5 XLM in base reserve, increasing the minimum balance.
- Trustlines include a limit (maximum balance) and track current balance and liabilities.

## Why Trustlines Matter

- Authorization: allow holding a specific asset (e.g., USDC from its issuer).
- Reserves: each trustline consumes XLM reserve to prevent abuse.
- Limits: set the maximum amount the account is willing to hold.

## ACTA Adaptation

- Price Per Credential: ACTA charges 1 USDC per issued credential.
- Required Trustlines: participants interacting with credential flows must have a USDC trustline established on the selected network.
- USDC Issuers (from SDK defaults):
  - Mainnet: `CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75` (`sdk/src/types/types.ts:17-20`)
  - Testnet: `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA` (`sdk/src/types/types.ts:21-24`)

## Practical Impact

- Before issuing or storing a credential, ensure all participating accounts have the USDC trustline set.
- The trustline must be active on the same network used by your dApp configuration (`mainNet` or `testNet`).

## Next Steps

- Obtain testnet XLM and USDC: see `docs/stellar-network/trustlines/testnet-tokens.md`.
- Connect a wallet and sign transactions: see `docs/developer-guide/wallet-kit-quick-integration.md`.