# Documentos

## Objeto Documento

```json title="Objeto Documento"
{
  "id": 4597246,
  "id_cliente": 123445,
  "no_documento": "API-0000001",
  "fecha_emision": "2026-03-03 00:00:00",
  "fecha_vencimiento": "2026-03-10 00:00:00",
  "url_ride": "https://ejemplo.com/ride/API-0000001.pdf",
  "valor_neto": 1.0,
  "valor_imp": 0.15,
  "saldo": 1.15,
  "tipo_documento": "FAC",
  "estado": 1,
  "informacion": "Documento creado via API",
  "cobros": []
}
```

### Estados del documento

El campo `estado` se maneja como un código numérico.

| `estado`  |
|-----------|
| COBRADO   |
| PENDIENTE |
| ANULADO   |

### Tipos de documento (`tipo_documento`)

Internamente, Pronto mapea el valor de `tipo_documento` a un código numérico. Los valores soportados son:

| `tipo_documento` |
|------------------|
| `FAC`            |
| `DNA`            |
| `EXP`            |

### Atributos del objeto Documento

| Parámetro           | Tipo     | Longitud | Obligatorio | Descripción                                            |
|---------------------|----------|----------|-------------|--------------------------------------------------------|
| `id`                | integer  |          | No          | Identificador interno del documento                    |
| `id_cliente`        | integer  |          | Si          | Identificador del cliente                              |
| `no_documento`      | string   | 50       | Si          | Referencia o número del documento                      |
| `fecha_emision`     | datetime |          | Si          | Fecha de emisión del documento (`YYYY-MM-DD HH:MM:SS`) |
| `fecha_vencimiento` | datetime |          | Si          | Fecha de vencimiento (`YYYY-MM-DD HH:MM:SS`)           |
| `url_ride`          | string   | 200      | No          | URL del RIDE/archivo del documento                     |
| `valor_neto`        | number   |          | Si          | Valor neto (sin impuestos)                             |
| `valor_imp`         | number   |          | Si          | Valor de impuestos                                     |
| `saldo`             | number   |          | Si          | Saldo pendiente del documento                          |
| `tipo_documento`    | string   | 3        | Si          | Tipo de documento (ejemplo: `FAC`)                     |
| `estado`            | integer  |          | Si          | Estado del documento (`0` a `5`)                       |
| `informacion`       | string   |          | No          | Información adicional (observación)                    |
| `cobros`            | list     |          | No          | Lista de pagos asociados (generalmente solo lectura)   |

## Obtener listado de Documentos (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/documents/
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
| `search`      | string  | Texto de búsqueda (según configuración del partner) |
| `fecha_desde` | date    | Filtra desde la fecha (`YYYY-MM-DD`)                |
| `fecha_hasta` | date    | Filtra hasta la fecha (`YYYY-MM-DD`)                |

### Respuesta

```json title="Respuesta paginada"
{
  "count": 2,
  "next": "https://app.conpronto.com/api/v1/documents/?p=2&page_size=10",
  "previous": null,
  "results": [
    {
      "id": 4597246,
      "id_cliente": 123445,
      "no_documento": "API-0000001",
      "fecha_emision": "2026-03-03 00:00:00",
      "fecha_vencimiento": "2026-03-10 00:00:00",
      "valor_neto": 1.0,
      "valor_imp": 0.15,
      "saldo": 1.15,
      "tipo_documento": "FAC",
      "estado": 1,
      "informacion": "Documento creado via API"
    }
  ]
}
```

## Obtener un Documento (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/documents/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

Respuesta 200
[Devuelve el objeto Documento.](#objeto-documento)

## Crear Documento (POST)

```bash title="URL"
POST https://app.conpronto.com/api/v1/documents/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body"
{
  "id_cliente": 123445,
  "no_documento": "API-0000001",
  "fecha_emision": "2026-03-03 00:00:00",
  "fecha_vencimiento": "2026-03-10 00:00:00",
  "url_ride": "https://ejemplo.com/ride/API-0000001.pdf",
  "valor_neto": 1.0,
  "valor_imp": 0.15,
  "saldo": 1.15,
  "tipo_documento": "FAC",
  "estado": 1,
  "informacion": "Documento creado via API",
  "cobros": [
    {
      "fecha_emision": "2026-03-10 10:00:00",
      "metodo_pago": "TRANSFERENCIA",
      "valor": 1.15
    }
  ]
}
```

### Comportamiento

- `id_cliente` debe existir y pertenecer al partner autenticado.
- Se recomienda inicializar `saldo` con el valor pendiente del documento (por ejemplo, `valor_neto + valor_imp`).
- `cobros` normalmente se envía como `[]` u omitir; los pagos se gestionan en el módulo de [Pagos](d_cobros.md).

### Respuesta

Respuesta 201
[Devuelve el objeto Documento.](#objeto-documento)

## Actualizar Documento (PUT)

```bash title="URL"
PUT https://app.conpronto.com/api/v1/documents/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body"
{
  "fecha_vencimiento": "2026-04-15 00:00:00",
  "url_ride": "https://ejemplo.com/ride/actualizado.pdf",
  "informacion": "Actualizado via API"
}
```

### Comportamiento

- Actualiza el documento indicado por `{id}`.

### Respuesta

Respuesta 200
[Devuelve el objeto Documento.](#objeto-documento)

## Eliminar Documento (DELETE)

```bash title="URL"
DELETE https://app.conpronto.com/api/v1/documents/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Comportamiento

- Elimina el documento indicado por `{id}`.
- Si el documento tiene pagos asociados o restricciones de negocio, la API eliminará rechazar la eliminación.

### Respuesta

```bash title="Respuesta"
204 No Content
```
