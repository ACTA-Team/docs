# useVaultApi

Hook for direct Vault contract reads and verification.

## Functions

```ts
listVcIdsDirect(args: { owner: string; vaultContractId?: string }): Promise<string[]>

getVcDirect(args: { owner: string; vcId: string; vaultContractId?: string }): Promise<unknown | null>

verifyInVault(args: { owner: string; vcId: string; vaultContractId?: string }): Promise<{ vc_id: string; status: string; since?: string }>
```

## Usage

```ts
import { useVaultApi } from "@acta-team/acta-sdk";

const { listVcIdsDirect, getVcDirect, verifyInVault } = useVaultApi();

const ids = await listVcIdsDirect({ owner, vaultContractId });
const vc = await getVcDirect({ owner, vcId, vaultContractId });
const verification = await verifyInVault({ owner, vcId, vaultContractId });
```