# useCreateCredential

Hook to issue a new credential via the API.

## Function

```ts
createCredential(payload: CreateCredentialPayload): Promise<CreateCredentialResponse>
```

## Payload Variants

- Signed XDR flow:
  - `{ signedXdr: string; vcId: string }`
- Server-issued flow:
  - `{ owner: string; vcId: string; vcData: string; vaultContractId: string; didUri?: string }`

## Return Value

- Resolves to the API response for credential creation.

## Usage

```ts
import { useCreateCredential } from "@acta-team/acta-sdk";

const { createCredential } = useCreateCredential();
const data = await createCredential({ signedXdr, vcId });
```