# useVaultStore

Hook to submit a signed XDR to store a credential in the Vault.

## Function

```ts
vaultStore(payload: { signedXdr: string; vcId: string; owner?: string; vaultContractId?: string }): Promise<{
  vc_id: string;
  tx_id: string;
  issue_tx_id?: string;
  verification?: { status?: string; since?: string };
}>
```

## Arguments

- `signedXdr`: signed transaction XDR string.
- `vcId`: credential identifier.
- `owner` (optional): owner public key.
- `vaultContractId` (optional): Vault contract ID.

## Return Value

- Store result including `vc_id`, `tx_id`, optional `issue_tx_id`, and optional `verification`.

## Usage

```ts
import { useVaultStore } from "@acta-team/acta-sdk";

const { vaultStore } = useVaultStore();
const result = await vaultStore({ signedXdr, vcId, owner, vaultContractId });
```