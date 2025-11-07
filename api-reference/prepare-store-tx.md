# POST /tx/prepare/store

Prepara un XDR sin firmar para `store_vc` usando campos normales. Pensado para flujos cliente‑firmados.

- Método: `POST`
- URL: `https://api.acta.build/tx/prepare/store`
- Content-Type: `application/json`

Cuerpo de solicitud:
```json
{
  "owner": "G...",
  "vcId": "vc:example:123",
  "didUri": "did:pkh:stellar:testnet:G...",
  "vaultContractId": "C...", // opcional
  "issuer": "G...",           // opcional
  "fields": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "type": "Attestation",
    "expires": "2025-12-31"
  }
}
```

Respuesta (200):
```json
{ "unsignedXdr": "AAAAAgAAAAA...==" }
```

Errores:
- 400 `owner_required` | `vcId_required` | `didUri_required` — faltan campos requeridos.
- 400 `bad_request` — detalles de error de simulación.
- 500 `prepare_error` — error interno.

Notas de uso:
- El servidor usa `ACTA_VAULT_CONTRACT_ID` cuando no se envía `vaultContractId`.
- Firma `unsignedXdr` con tu wallet y envíalo a `POST /vault/store` como `signedXdr`.