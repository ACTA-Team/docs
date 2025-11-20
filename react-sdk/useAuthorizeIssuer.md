# useAuthorizeIssuer

Hook to authorize an issuer address in the Vault contract by building, signing, submitting, and waiting for the transaction.

## Function

```ts
authorizeIssuer(args: {
  owner: string;
  issuer: string;
  signTransaction: (unsignedXdr: string, opts: { networkPassphrase: string }) => Promise<string>;
}): Promise<{ txId: string }>
```

## Arguments

- `owner`: Stellar public key (G...).
- `issuer`: Stellar public key to authorize as issuer.
- `signTransaction`: function that signs the unsigned XDR with the given `networkPassphrase`.

## Return Value

- `{ txId: string }`: transaction hash after sending to the network.

## Usage

```ts
import { useAuthorizeIssuer } from "@acta-team/acta-sdk";

const { authorizeIssuer } = useAuthorizeIssuer();
const { txId } = await authorizeIssuer({ owner, issuer, signTransaction });
```