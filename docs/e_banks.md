# Cuentas Bancarias

Las cuentas bancarias permiten configurar las cuentas asociadas al partner autenticado.

Estas cuentas son utilizadas para procesos como:

- Registro de pagos por transferencia (`TRA`)
- Registro de pagos por cheque (`CHQ`)
- Débitos automáticos
- Conciliación bancaria

---

# Bancos

## Objeto Banco

```json title="Objeto Banco"
{
  "id_banco": 0,
  "nombre_banco": "Banco Amazonas"
}
```

## Atributos del objeto Banco

| Parámetro      | Tipo    | Longitud | Obligatorio | Descripción                     |
|----------------|---------|----------|-------------|---------------------------------|
| `id_banco`     | integer |          | No          | Identificador interno del banco |
| `nombre_banco` | string  | 100      | No          | Nombre del banco                |

## Obtener listado de Bancos (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/banks/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Respuesta

```json title="Respuesta"
{
  "estado": true,
  "mensaje": "Bancos consultados correctamente",
  "datos": [
    {
      "id_banco": 0,
      "nombre_banco": "Banco Amazonas"
    }
  ]
}
```

---

# Cuentas Bancarias

## Objeto Cuenta Bancaria

```json title="Objeto Cuenta Bancaria"
{
  "id_cuenta": 0,
  "id_banco": 0,
  "nombre_banco": "Banco Bolivariano",
  "numero_cuenta": "555555555",
  "beneficiario": "EMPRESA ACTUALIZADA",
  "email": "nuevo@empresa.com",
  "identificacion": "0999999999001",
  "tipo_cuenta": 1,
  "tipo_cuenta_descripcion": "CUENTA AHORROS",
  "codigo_establecimiento": "002",
  "convenio": "999",
  "cuenta_debitos_automaticos": true,
  "disponible_conciliacion": true,
  "id_contifico": null,
  "id_externo": "EXT-002"
}
```

## Tipos de Cuenta (`tipo_cuenta`)

| tipo_cuenta | Descripción      |
|-------------|------------------|
| 0           | CUENTA CORRIENTE |
| 1           | CUENTA AHORROS   |

## Atributos del objeto Cuenta Bancaria

| Parámetro                    | Tipo    | Longitud | Obligatorio | Descripción                                                 |
|------------------------------|---------|----------|-------------|-------------------------------------------------------------|
| `id_cuenta`                  | integer |          | No          | Identificador interno de la cuenta                          |
| `id_banco`                   | integer |          | Si          | Banco asociado                                              |
| `nombre_banco`               | string  |          | No          | Nombre del banco asociado                                   |
| `numero_cuenta`              | string  | 30       | Si          | Número de cuenta bancaria                                   |
| `beneficiario`               | string  | 200      | No          | Nombre del beneficiario                                     |
| `email`                      | string  | 200      | No          | Correo asociado                                             |
| `identificacion`             | string  | 22       | No          | RUC o identificación del beneficiario                       |
| `tipo_cuenta`                | integer |          | Si          | Tipo de cuenta bancaria                                     |
| `tipo_cuenta_descripcion`    | string  |          | No          | Descripción del tipo de cuenta                              |
| `codigo_establecimiento`     | string  |          | No          | Código de establecimiento                                   |
| `convenio`                   | string  |          | No          | Convenio bancario                                           |
| `cuenta_debitos_automaticos` | boolean |          | No          | Indica si la cuenta será utilizada para débitos automáticos |
| `disponible_conciliacion`    | boolean |          | No          | Indica si está disponible para conciliación                 |
| `id_externo`                 | string  |          | No          | Identificador externo de integración                        |

---

# Obtener listado de Cuentas Bancarias (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/bank_accounts/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Respuesta

```json title="Respuesta"
{
  "estado": true,
  "mensaje": "Cuentas bancarias consultadas correctamente",
  "datos": [
    {
      "id_cuenta": 0,
      "id_banco": 0,
      "nombre_banco": "Banco Bolivariano",
      "numero_cuenta": "555555555",
      "beneficiario": "EMPRESA ACTUALIZADA",
      "email": "nuevo@empresa.com",
      "identificacion": "0999999999001",
      "tipo_cuenta": 1,
      "tipo_cuenta_descripcion": "CUENTA AHORROS",
      "cuenta_debitos_automaticos": true,
      "disponible_conciliacion": true,
      "id_externo": "EXT-002"
    }
  ]
}
```

---

# Crear Cuenta Bancaria (POST)

```bash title="URL"
POST https://app.conpronto.com/api/v1/bank_accounts/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

## Body

```json
{
  "id_banco": 0,
  "numero_cuenta": "0123456789",
  "beneficiario": "EMPRESA PRUEBA S.A.",
  "email": "pagos@empresa.com",
  "identificacion": "0999999999001",
  "tipo_cuenta": 0,
  "codigo_establecimiento": "001",
  "convenio": "123456",
  "cuenta_debitos_automaticos": true,
  "disponible_conciliacion": false,
  "id_externo": "EXT-001"
}
```

## Validaciones

- `id_banco` debe existir en el catálogo de bancos.
- La cuenta bancaria queda asociada automáticamente al partner autenticado.
- Solo puede existir una cuenta con:

```json
{
  "cuenta_debitos_automaticos": true
}
```

por partner.

### Respuesta

```json
{
  "estado": true,
  "mensaje": "Cuenta bancaria creada correctamente",
  "datos": {
    "id_cuenta": 1273,
    "numero_cuenta": "0123456789"
  }
}
```

---

# Actualizar Cuenta Bancaria (PUT)

```bash title="URL"
PUT https://app.conpronto.com/api/v1/bank_accounts/{id}/
```

## Body

```json
{
  "id_banco": 6,
  "numero_cuenta": "555555555",
  "beneficiario": "EMPRESA ACTUALIZADA",
  "email": "nuevo@empresa.com",
  "identificacion": "0999999999001",
  "tipo_cuenta": 1,
  "codigo_establecimiento": "002",
  "convenio": "999",
  "cuenta_debitos_automaticos": true,
  "disponible_conciliacion": true,
  "id_externo": "EXT-002"
}
```

### Respuesta

```json
{
  "estado": true,
  "mensaje": "Cuenta bancaria actualizada correctamente",
  "datos": {
    "id_cuenta": 1273,
    "numero_cuenta": "555555555"
  }
}
```

---

# Eliminar Cuenta Bancaria (DELETE)

```bash title="URL"
DELETE https://app.conpronto.com/api/v1/bank_accounts/{id}/
```

### Respuesta

```json
{
  "estado": true,
  "mensaje": "Cuenta bancaria eliminada correctamente"
}
```