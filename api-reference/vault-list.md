# POST /vault/list_vc_ids

Ejecuta una lectura firmada para listar todos los IDs de VC del Vault del propietario.

- Method: `POST`
- URLs:
  - Testnet: `https://api.testnet.acta.build/vault/list_vc_ids`
  - Mainnet: `https://api.mainnet.acta.build/vault/list_vc_ids`
- Content-Type: `application/json`

Cuerpo de solicitud:
```json
{
  "signedXdr": "AAAAAgAAAAA...=="
}
```

Éxito (200):
```json
{ "vc_ids": ["vc:example:acta", "http://university.example/credentials/3732"] }
```

Errores:
- 400 `signed_xdr_required` — falta el sobre firmado.
- 500 `read_error` — error interno o forma de retorno inesperada.

Notas de uso:
- Usa `POST /tx/prepare/list_vc_ids` para obtener el `unsignedXdr`, fírmalo con el wallet y envíalo aquí como `signedXdr`.
- Esta llamada requiere firma del `owner` por `require_auth` en el contrato Vault.