# API Reference

Base URL: `https://api.acta.build`

Endpoints:
- `GET /health` — service status and environment info
- `POST /credentials` — issue a VC and store it in the Vault
- `POST /vault/store` — store a VC in the Vault (server-signed)
- `POST /vault/list_vc_ids` — list VC IDs (client-signed read)
- `POST /vault/get_vc` — fetch a VC by ID (client-signed read)
- `GET /verify/{vc_id}` — verify VC status
 

See detailed pages for payloads, responses, and example requests.