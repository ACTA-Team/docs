# Troubleshooting & FAQs

Common issues and answers adapted to ACTA.

## Common Issues

- Unable to connect wallet to ACTA
  - Ensure your wallet extension (e.g., Freighter/Albedo) is installed and enabled
  - Verify wallet network matches your app provider configuration: `ActaConfig baseURL` set to `mainNet` or `testNet` (`sdk/src/index.ts:12-14`)
  - If using a signer, ensure the correct `networkPassphrase` from `client.getDefaults()` (`sdk/src/client.ts:55-63`)

- Error: Insufficient Balance
  - Check XLM base reserve requirements for the account and any trustlines (each trustline adds ~0.5 XLM to minimum balance)
  - Ensure you have sufficient XLM for fees on testnet/mainnet
  - ACTA price per credential: 1 USDC — confirm USDC trustline and balance ≥ 1 USDC
  - USDC issuers:
    - Mainnet: `CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75` (`sdk/src/types/types.ts:17-20`)
    - Testnet: `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA` (`sdk/src/types/types.ts:21-24`)

- Error: `issuer_not_authorized`
  - The issuer is not authorized in the owner’s Vault
  - Authorize using the SDK hook `useAuthorizeIssuer` before issuing/storing
  - Error reference: `api/testnet/src/controllers/credentialsController.ts:52-57`, `api/testnet/src/controllers/vaultStoreController.ts:87-91`

- Error: `signed_xdr_invalid`
  - The signed XDR is not valid for the selected network or payload
  - Ensure the wallet signs with the expected `networkPassphrase` and correct XDR
  - Error reference: `api/testnet/src/controllers/credentialsController.ts:19-22`, `api/testnet/src/controllers/vaultStoreController.ts:29-33`

- Error: `bad_request` (simulation)
  - Transaction simulation failed; validate payload fields and contract IDs
  - References: `api/testnet/src/controllers/txPrepareStoreController.ts:41-45`, `txPrepareIssueController.ts:23-27`, `vaultStoreController.ts:93-97`

- Validation errors
  - `owner_required`, `vcId_required`, `vcData_required`, `vaultContractId_required`, `didUri_required`, `issuer_required`
  - References: `api/testnet/src/controllers/*.ts`

## FAQs

- How do I switch networks?
  - Toggle wallet network (Testnet/Mainnet)
  - Set `ActaConfig baseURL` to `mainNet` or `testNet` (`sdk/src/index.ts:12-14`)

- Can I recover my wallet without the recovery phrase?
  - No. Always back up your seed phrase securely

- How do I set the USDC trustline?
  - Follow `docs/stellar-network/trustlines/index.md` and use the issuer IDs above

- How do I sign transactions correctly?
  - Use a signer `(unsignedXdr: string, opts: { networkPassphrase: string }) => Promise<string>`
  - Obtain `networkPassphrase` from `client.getDefaults()` and pass it to the wallet kit signer (`sdk/src/client.ts:55-63`)
