# useTxPrepare

Hook to prepare unsigned XDRs for store, issue, list VC IDs, and get VC flows.

## Functions

```ts
prepareStore(args: {
  owner: string;
  vcId: string;
  didUri: string;
  fields: Record<string, unknown>;
  vaultContractId?: string;
  issuer?: string;
}): Promise<{ unsignedXdr: string }>

prepareIssue(args: {
  owner: string;
  vcId: string;
  vcData: string;
  vaultContractId?: string;
  issuer?: string;
  issuerDid?: string;
}): Promise<{ unsignedXdr: string }>

prepareListVcIds(args: { owner: string; vaultContractId?: string }): Promise<{ unsignedXdr: string }>

prepareGetVc(args: { owner: string; vcId: string; vaultContractId?: string }): Promise<{ unsignedXdr: string }>
```

## Usage

```ts
import { useTxPrepare } from "@acta-team/acta-sdk";

const { prepareStore, prepareIssue, prepareListVcIds, prepareGetVc } = useTxPrepare();

const { unsignedXdr: storeXdr } = await prepareStore({ owner, vcId, didUri, fields, vaultContractId, issuer });
const { unsignedXdr: issueXdr } = await prepareIssue({ owner, vcId, vcData, vaultContractId, issuer, issuerDid });
const { unsignedXdr: listXdr } = await prepareListVcIds({ owner, vaultContractId });
const { unsignedXdr: getXdr } = await prepareGetVc({ owner, vcId, vaultContractId });
```