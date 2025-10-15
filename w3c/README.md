# W3C Alignment

The ACTA API is designed to interoperate with W3C specifications relevant to Verifiable Credentials (VCs) and Decentralized Identifiers (DIDs).

## Verifiable Credentials

- `vcData` expects a stringified JSON object that follows the VC data model conventions.
- `vcId` identifies the credential (e.g., `vc:example:123`).
- Storage is performed on-chain through Smart Contracts (Soroban), providing tamper-evident records.

## DIDs

- `didUri` (optional) uses the W3C DID URI format: `did:<method>:<id>`.
- DIDs created via `https://keys.acta.build` associate the credential owner with a decentralized identity.

## Verification

- `GET /verify/{vc_id}` returns VC status: `valid`, `invalid`, or `revoked`.
- The verification process reads state from the issuance contract and maps it into the VC status model.