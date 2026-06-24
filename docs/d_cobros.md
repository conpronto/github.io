# Pagos

## Objeto Pago

```json title="Objeto Pago"
{
  "id": 981234,
  "id_cliente": 123445,
  "fecha_emision": "2026-03-10 10:00:00",
  "metodo_pago": "TRA",
  "numero_comprobante": "TRX-001245",
  "id_cuenta_bancaria": 1273,
  "total": 4.05,
  "documentos": [
    {
      "id_documento": 4597246,
      "valor": 1.15
    },
    {
      "id_documento": 3768142,
      "valor": 0.6
    },
    {
      "id_documento": 3425488,
      "valor": 2.3
    }
  ]
}
```

### Atributos del objeto Pago

| Parámetro              | Tipo     | Longitud  | Obligatorio   | Descripción                                                    |
|------------------------|----------|-----------|---------------|----------------------------------------------------------------|
| `id`                   | integer  |           | No            | Identificador interno del pago                                 |
| `id_cliente`           | integer  |           | Si            | Identificador del cliente                                      |
| `fecha_emision`        | datetime |           | Si            | Fecha/hora del pago (`YYYY-MM-DD HH:MM:SS`)                    |
| `metodo_pago`          | string   | 3         | Si            | Método de pago utilizado                                       |
| `numero_comprobante`   | string   | 150       | Solo TRA/CHQ  | Número de comprobante bancario, transferencia o cheque         |
| `id_cuenta_bancaria`   | integer  |           | Solo TRA/CHQ  | Identificador de la cuenta bancaria donde se registra el cobro |
| `total`                | number   |           | Si            | Valor total del pago                                           |
| `documentos`           | list     |           | Si            | Lista de documentos a los que aplica el pago                   |

#### Atributos del objeto `documentos[]`

| Parámetro      | Tipo    | Longitud | Obligatorio | Descripción                             |
|----------------|---------|----------|-------------|-----------------------------------------|
| `id_documento` | integer |          | Si          | Identificador del documento             |
| `valor`        | number  |          | Si          | Valor del pago aplicado a ese documento |

### Métodos de pago (`metodo_pago`)

Internamente, Pronto mapea el valor de `metodo_pago` a un código numérico.

Valores soportados:

| `metodo_pago` | Descripción   |
|---------------|---------------|
| `EF`          | Efectivo      |
| `TRA`         | Transferencia |
| `TC`          | Tarjeta       |
| `CHQ`         | Cheque        |

### Validación para Transferencias y Cheques

Cuando el método de pago sea:

- `TRA` (Transferencia)
- `CHQ` (Cheque)

Los siguientes campos son obligatorios:

| Parámetro            | Descripción                                                   |
|----------------------|---------------------------------------------------------------|
| `numero_comprobante` | Número de referencia bancaria, comprobante o número de cheque |
| `id_cuenta_bancaria` | Cuenta bancaria donde se registra el pago                     |

Si alguno de estos campos no es enviado, el pago será rechazado.

La cuenta bancaria enviada debe existir y pertenecer al partner autenticado.

## Obtener listado de Pagos (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/payments/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Parámetros de Consulta (Opcionales)

| Parámetro     | Tipo    | Descripción                                         |
|---------------|---------|-----------------------------------------------------|
| `p`           | integer | Número de página                                    |
| `page_size`   | integer | Cantidad de registros por página                    |
| `id_cliente`  | integer | Filtra por cliente                                  |
| `metodo_pago` | string  | Filtra por método de pago                           |
| `search`      | string  | Texto de búsqueda (según configuración del partner) |
| `fecha_desde` | date    | Filtra desde la fecha (`YYYY-MM-DD`)                |
| `fecha_hasta` | date    | Filtra hasta la fecha (`YYYY-MM-DD`)                |

### Respuesta

```json title="Respuesta paginada"
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 981234,
      "id_cliente": 123445,
      "fecha_emision": "2026-03-10 10:00:00",
      "metodo_pago": "TRA",
      "numero_comprobante": "TRX-001245",
      "id_cuenta_bancaria": 1273,
      "total": 4.05
    }
  ]
}
```

## Obtener un Pago (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/payments/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

Respuesta 200
[Devuelve el objeto Pago.](#objeto-pago)

## Crear Pago (POST)

```bash title="URL"
POST https://app.conpronto.com/api/v1/payments/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body"
{
  "id_cliente": 123445,
  "fecha_emision": "2026-03-10 10:00:00",
  "metodo_pago": "TRA",
  "numero_comprobante": "TRX-001245",
  "id_cuenta_bancaria": 1273,
  "total": 4.05,
  "documentos": [
    {
      "id_documento": 4597246,
      "valor": 1.15
    },
    {
      "id_documento": 3768142,
      "valor": 0.6
    },
    {
      "id_documento": 3425488,
      "valor": 2.3
    }
  ]
}
```

### Comportamiento

- `id_cliente` debe existir y pertenecer al partner autenticado.
- Cada item de `documentos[]` aplica un valor de pago a un documento.
- La suma de los valores enviados en `documentos[]` debe coincidir con el campo `total`.
- El pago actualizará el saldo de los documentos relacionados.
- Para métodos de pago:
  - `TRA` (Transferencia)
  - `CHQ` (Cheque)

  es obligatorio enviar:

  - `numero_comprobante`
  - `id_cuenta_bancaria`

- La cuenta bancaria debe existir y pertenecer al partner autenticado.
- Métodos como `EF` y `TC` no requieren información bancaria.

### Respuesta

Respuesta 201
[Devuelve el objeto Pago.](#objeto-pago)

## Eliminar Pago (DELETE)

```bash title="URL"
DELETE https://app.conpronto.com/api/v1/payments/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Respuesta

```bash title="Respuesta"
204 No Content
```
