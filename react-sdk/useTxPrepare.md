# useTxPrepare

Hook to prepare unsigned XDRs for various vault and issuance operations.

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

prepareInitialize(args: {
  owner: string;
  didUri: string;
  vaultContractId?: string;
  sourcePublicKey: string;
}): Promise<{ xdr: string; network: string }>

prepareAuthorizeIssuer(args: {
  owner: string;
  issuer: string;
  vaultContractId?: string;
  sourcePublicKey: string;
}): Promise<{ xdr: string; network: string }>

prepareAuthorizeIssuers(args: {
  owner: string;
  issuers: string[];
  vaultContractId?: string;
  sourcePublicKey: string;
}): Promise<{ xdr: string; network: string }>

preparePush(args: {
  fromOwner: string;
  toOwner: string;
  vcId: string;
  issuer: string;
  vaultContractId?: string;
  sourcePublicKey: string;
}): Promise<{ xdr: string; network: string }>

prepareRevokeVault(args: {
  owner: string;
  vaultContractId?: string;
  sourcePublicKey: string;
}): Promise<{ xdr: string; network: string }>
```

## Usage

```ts
import { useTxPrepare } from "@acta-team/acta-sdk";

const { 
  prepareStore, 
  prepareIssue, 
  prepareInitialize,
  prepareAuthorizeIssuer,
  prepareAuthorizeIssuers,
  preparePush,
  prepareRevokeVault
} = useTxPrepare();

// Prepare store transaction
const { unsignedXdr: storeXdr } = await prepareStore({ 
  owner, 
  vcId, 
  didUri, 
  fields, 
  vaultContractId, 
  issuer 
});

// Prepare issue transaction
const { unsignedXdr: issueXdr } = await prepareIssue({ 
  owner, 
  vcId, 
  vcData, 
  vaultContractId, 
  issuer, 
  issuerDid 
});

// Prepare initialize transaction
const { xdr: initXdr, network } = await prepareInitialize({
  owner,
  didUri,
  vaultContractId,
  sourcePublicKey
});

// Prepare authorize issuer transaction
const { xdr: authXdr, network } = await prepareAuthorizeIssuer({
  owner,
  issuer,
  vaultContractId,
  sourcePublicKey
});

// Prepare push transaction
const { xdr: pushXdr, network } = await preparePush({
  fromOwner,
  toOwner,
  vcId,
  issuer,
  vaultContractId,
  sourcePublicKey
});
```

## Notes

- Read-only operations (`list_vc_ids`, `get_vc`, `verify_vc`) do not require transaction preparation and are available directly via `useVaultApi`
- All prepare functions return XDR strings that must be signed by the client
- Signed XDRs are submitted to the corresponding endpoint (e.g., signed initXdr → `/vault/initialize`)
