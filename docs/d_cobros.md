# Pagos 

En esta documentación, un **pago** corresponde a lo que históricamente se conocía como **cobro**.

## Objeto Pago

```json title="Objeto Pago"
{
  "id": 981234,
  "id_cliente": 123445,
  "fecha_emision": "2026-03-10 10:00:00",
  "metodo_pago": "TRANSFERENCIA",
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

| Parámetro       | Tipo     | Longitud | Obligatorio | Descripción |
|----------------|----------|----------|-------------|-------------|
| `id`           | integer  |          | No          | Identificador interno del pago |
| `id_cliente`   | integer  |          | Si          | Identificador del cliente |
| `fecha_emision`| datetime |          | Si          | Fecha/hora del pago (`YYYY-MM-DD HH:MM:SS`) |
| `metodo_pago`  | string   |          | Si          | Método de pago (ejemplo: `TRANSFERENCIA`) |
| `total`        | number   |          | Si          | Valor total del pago |
| `documentos`   | list     |          | Si          | Lista de documentos a los que aplica el pago |

#### Atributos del objeto `documentos[]`

| Parámetro       | Tipo    | Longitud | Obligatorio | Descripción |
|----------------|---------|----------|-------------|-------------|
| `id_documento` | integer |          | Si          | Identificador del documento |
| `valor`        | number  |          | Si          | Valor del pago aplicado a ese documento |

### Métodos de pago (`metodo_pago`)

Internamente, Pronto mapea el valor de `metodo_pago` a un código numérico. Los valores soportados son:

| `metodo_pago`        | Código |
|----------------------|--------|
| `EFECTIVO`           | `0`    |
| `TRANSFERENCIA`      | `1`    |
| `TARJETA CREDITO`    | `2`    |
| `TARJETA CRÉDITO`    | `2`    |
| `CHEQUE`             | `3`    |
| `CRUCE DE DOCUMENTO` | `4`    |

## Obtener listado de Pagos (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/payments/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Parámetros de Consulta (Opcionales)

| Parámetro     | Tipo    | Descripción |
|---------------|---------|-------------|
| `p`           | integer | Número de página |
| `page_size`   | integer | Cantidad de registros por página |
| `id_cliente`  | integer | Filtra por cliente |
| `metodo_pago` | string  | Filtra por método de pago |
| `search`      | string  | Texto de búsqueda (según configuración del partner) |
| `fecha_desde` | date    | Filtra desde la fecha (`YYYY-MM-DD`) |
| `fecha_hasta` | date    | Filtra hasta la fecha (`YYYY-MM-DD`) |

### Respuesta

Cuando `paginado` no se envía (o es `true`), la respuesta es paginada:

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
      "metodo_pago": "TRANSFERENCIA",
      "total": 4.05
    }
  ]
}
```

Cuando `paginado=false`, la respuesta es una lista:

```json title="Respuesta (paginado=false)"
[
  {
    "id": 981234,
    "id_cliente": 123445,
    "fecha_emision": "2026-03-10 10:00:00",
    "metodo_pago": "TRANSFERENCIA",
    "total": 4.05
  }
]
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
  "metodo_pago": "TRANSFERENCIA",
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
