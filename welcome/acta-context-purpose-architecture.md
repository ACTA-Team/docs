# ACTA: Context, Purpose, and Architecture (Full Summary)

## 1) What is ACTA? (Elevator Pitch)

ACTA is a 100% on‑chain verifiable credentials infrastructure on Stellar, built to eliminate fake certificates and enable instant, secure, public verification without centralized servers. It issues W3C Verifiable Credentials 2.0, anchors their hash on Soroban, and stores encrypted VC data in the holder’s Vault — driven only by cryptographic keys.

Built for startups integrating trust in hours via API/SDK and templates.

## 2) Problem We Solve

Paper/PDF credentials:
- Easy to fake
- Hard to verify without contacting the issuer
- Lack cryptographic protection and public state

Existing VC stacks (Polygon ID, Veramo, Ceramic, Spruce, etc.) can be complex or rely on heavy off‑chain infra. No native, full on‑chain VC system existed on Stellar.

## 3) Value Proposition

For Startups
- Fast issuance of verifiable credentials
- Lower fraud via public verification
- Simple REST API and SDK integrations

For Users
- A digital passport of achievements
- Encrypted storage in their personal Vault
- Data control and portability

For Verifiers
- Instant confirmation via on‑chain hash and API reads
- No dependence on third parties

## 4) System Architecture (Simple and Powerful)

### 4.1 Identity
- DID:pkh:stellar — `did:pkh:stellar:<wallet>`
- The user’s Stellar wallet is their identity; no extra accounts required

### 4.2 Credential Issuance
- Apps define a schema
- ACTA API receives VC data, canonicalizes, signs, and stores in the holder’s Vault
- The VC hash is anchored on Soroban for public verification
- Relevant endpoints: `POST /credentials`, `POST /vault/store`, `POST /vault/list_vc_ids` (see `docs/api-reference`)

### 4.3 Vault (W3C Repository)
- W3C‑aligned repository where holders store encrypted credentials
- Multiple VCs per user; holder controls access
- Confidentiality + Integrity principles aligned to W3C repository concepts
- Integrates with Stellar wallets
- Provisioned via contracts and encrypted buckets, facilitated by ACTA

### 4.4 Public API
- `POST /credentials` — issuer flow: validate, sign, store in Vault
- `POST /vault/store` — store a VC already issued by an external issuer
- `POST /vault/list_vc_ids` — list holder’s credential IDs
- `POST /vault/get_vc` — fetch VC contents
- `POST /verify` / `GET /verify/:vc_id` — verify status
- Full details in `docs/api-reference`

## 5) Technology
- Soroban (Stellar): public state and verification
- DID:pkh for identity binding
- Encrypted buckets (Supabase) for off‑chain storage
- HMAC signatures between integrators where applicable
- W3C VC 2.0, VC Data Model, JSON‑LD compatibility

## 6) Real Use Cases
- TrustlessWork — delivery certificates tied to escrow outcomes
- GrantFox — participation certificates, grant validations and tiers
- KindFi — certificates for donors and communities

General applications
- Diplomas and educational certificates
- Work evidence for freelancing
- Audit certificates
- Identity and access proofs

## 7) Future Vision
- Standardize identity and credentials across Stellar
- Wallet SDK for ACTA with selective disclosure and ZK‑compatible claims
- Admin panel for institutions to issue without code
- Multichain verification (cross‑chain, zk‑anchors)
- Native DID resolution for Stellar
- ZK proofs for statements like: “I’m over 18” (without revealing the date), “I hold a valid credential for X” (without disclosing the document)