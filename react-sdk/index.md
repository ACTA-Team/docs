# React SDK Overview

React library exposing a provider, client access, and hooks for ACTA API and Soroban transactions. The network is inferred from the `baseURL`.

## Exports

- `ActaConfig` provider and `useActaClient` context accessor
- Hooks: `useCreateCredential`, `useCreateVault`, `useAuthorizeIssuer`, `useVaultApi`, `useTxPrepare`, `useVaultStore`
- Base URLs: `mainNet` and `testNet`

## Provider Setup

```tsx
import { ActaConfig, mainNet } from "@acta-team/acta-sdk";

export function App() {
  return (
    <ActaConfig baseURL={mainNet} apiKey={"your-api-key"}>
      {/* your app */}
    </ActaConfig>
  );
}
```

## Accessing the Client

```ts
import { useActaClient } from "@acta-team/acta-sdk";

const client = useActaClient();
const defaults = client.getDefaults();
// defaults: { rpcUrl, networkPassphrase, issuanceContractId, vaultContractId }
```

## Hooks Summary

- `useCreateCredential`: issue a credential via the API
- `useCreateVault`: build, sign, and submit the `initialize` tx
- `useAuthorizeIssuer`: authorize an issuer in the Vault
- `useVaultApi`: direct Vault reads (list IDs, get VC) and verification
- `useTxPrepare`: build unsigned XDRs for store/issue/list/get flows
- `useVaultStore`: submit signed XDR to store a credential