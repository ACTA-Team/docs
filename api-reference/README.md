# API Reference

Base URL: `https://api.acta.build`

Endpoints:
- `GET /health` — service status and environment info
- `POST /credentials` — issue a VC and store it in the Vault
- `POST /vault/store` — store a VC in the Vault (owner-signed)
- `GET /verify/{vc_id}` — verify VC status
- Client-signed (optional): `POST /tx/prepare/issue`, `POST /tx/prepare/store`, `POST /tx/submit`

See detailed pages for payloads, responses, and example requests.