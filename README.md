# ACTA API Documentation

This GitBook documents the current state of the ACTA API and how companies can integrate it directly via the public endpoint `https://api.acta.build`.

Important: Before using the API, create your Vault and DID at `https://keys.acta.build`. The Vault is required for on-chain storage, and the DID associates ownership and identity per W3C standards.

## What This Covers

- Public endpoints and payloads
- Minimal W3C alignment notes (VCs and DIDs)
- How to integrate the API into your systems without cloning any repository

## Base URL

All endpoints are served under:

`https://api.acta.build`

## Quick Flow

1. Create a Vault and DID at `https://keys.acta.build` for your organization or end user.
2. Use the API to issue and/or store a Verifiable Credential (VC) into the Vault.
3. Verify VC status when needed via the verification endpoint.