# API Reference

Base URL: `https://api.acta.build`

Current endpoints:
- `GET /health` — service status and environment snapshot.
- `GET /config` — active configuration: `rpcUrl`, `networkPassphrase`, `issuanceContractId`, `vaultContractId`.
- `POST /credentials` — issue a credential and store it in the owner’s Vault (server‑signed).
- `POST /vault/store` — store directly in the Vault (server‑signed, issuer must be authorized).
- `POST /vault/list_vc_ids` — list VC IDs using `signedXdr` (client‑signed read).
- `POST /vault/get_vc` — fetch a VC by ID using `signedXdr` (client‑signed read).
- `POST /vault/list_vc_ids_direct` — list VC IDs without signatures, using `owner`.
- `POST /vault/get_vc_direct` — fetch a VC without signatures, using `owner` and `vcId`.
- `POST /vault/verify` — verify VC status via Vault using `{ owner, vcId }`.
- `POST /vault/revoke_issuer` — revoke an authorized issuer in a Vault (signedXdr or direct).
- `POST /tx/prepare/issue` — prepare unsigned XDR for issuing a VC.
- `POST /tx/prepare/store` — prepare unsigned XDR for `store_vc` in the Vault.
- `POST /tx/prepare/list_vc_ids` — prepare unsigned XDR for listing IDs.
- `POST /tx/prepare/get_vc` — prepare unsigned XDR for reading a VC.
- `GET /verify/{vc_id}` — verify VC status via Issuance contract.

See each endpoint page for exact payloads, responses, and examples.