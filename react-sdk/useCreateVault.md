# useCreateVault

Hook to initialize a Vault for an owner by building, signing, and submitting the `initialize` transaction.

## Function

```ts
createVault(args: {
  owner: string;
  ownerDid: string;
  signTransaction: (unsignedXdr: string, opts: { networkPassphrase: string }) => Promise<string>;
}): Promise<{ txId: string }>
```

## Arguments

- `owner`: Stellar public key (G...).
- `ownerDid`: DID string bound to the owner.
- `signTransaction`: function that signs the unsigned XDR with the given `networkPassphrase`.

## Return Value

- `{ txId: string }`: transaction hash after sending to the network.

## Usage

```ts
import { useCreateVault } from "@acta-team/acta-sdk";

const { createVault } = useCreateVault();
const { txId } = await createVault({ owner, ownerDid, signTransaction });
```