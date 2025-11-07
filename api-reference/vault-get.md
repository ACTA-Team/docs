# POST /vault/get_vc

Ejecuta una lectura firmada para obtener un VC específico del Vault del propietario.

- Method: `POST`
- URL: `https://api.acta.build/vault/get_vc`
- Content-Type: `application/json`

Cuerpo de solicitud:
```json
{
  "signedXdr": "AAAAAgAAAAA...=="
}
```

Éxito (200):
```json
{
  "vc": {
    "id": "vc:example:123",
    "issuer_did": "did:pkh:stellar:testnet:GABCDE...",
    "issuance_contract": "CANYEUDJCAPQ5AC...",
    "data": "{...}"
  }
}
```

Errores:
- 400 `signed_xdr_required` — falta el sobre firmado.
- 500 `read_error` — error interno.

Notas de uso:
- Usa `POST /tx/prepare/get_vc` para obtener el `unsignedXdr`, fírmalo con el wallet y envíalo aquí como `signedXdr`.
- Esta llamada requiere firma del `owner` por `require_auth` en el contrato Vault.