# Integration Guide

This guide explains how a company can integrate ACTA using only the public API at `https://api.acta.build`.

## Prerequisites

- Create your Vault and DID at `https://keys.acta.build`.
- Obtain the `owner` G-address and `vaultContractId` for your Vault.

## Typical Use Cases

- Issue and store a VC for a user or asset
- Verify VC status during onboarding or compliance checks

## Minimal Integration Steps

1. Prepare your VC JSON payload and choose a `vcId`.
2. Call `POST /credentials` to issue and store the VC in the Vault.
3. If you need owner-signed storage only, use `POST /vault/store` instead.
4. Verify when needed via `GET /verify/{vc_id}`.

## Example Requests

Issue:
```bash
curl -X POST https://api.acta.build/credentials \
  -H "Content-Type: application/json" \
  -d '{
    "owner": "G...",
    "vaultContractId": "C...",
    "vcId": "vc:example:123",
    "vcData": "{\"type\":\"Attestation\",\"subject\":\"...\"}",
    "didUri": "did:acta:owner-123"
  }'
```

Verify:
```bash
curl https://api.acta.build/verify/vc:example:123
```

## Handling Errors

- `403 issuer_not_authorized`: The issuer must be authorized in the owner’s Vault. Configure authorization via the Vault UI at `https://keys.acta.build` and retry.
- `400 bad_request`: Check the request payload and VC JSON. Fix validation or simulation errors.
- `5xx`: Retry later or contact support if persistent.

## Optional: Client-signed Workflows

If your security model requires client-side signing, use the prepare/submit endpoints described in the API Reference.

## Security Notes

- Always use HTTPS.
- Your `STELLAR_SECRET_KEY` stays within your infrastructure (e.g., `.env`, HSM, wallet kit).
- The ACTA API never requests, receives, or stores private keys.
- Treat transaction hashes (`tx_id`) and credential identifiers (`vc_id`) as sensitive operational data.