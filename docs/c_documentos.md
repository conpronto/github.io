# Documento
## Objeto Documento

``` json title="Objeto Documento:"
[{
    "id": "7466",
    "cliente_id": "52235",
    "suscripcion_id": "74135",
    "fecha_modificacion": "22/06/2023",
    "fecha_emision": "13/06/2023",
    "fecha_vencimiento": "17/10/2023",
    "numero": "001-001-000000251",
    "autorizacion": "45678932598466966",
    "estado": "1",
    "descripcion": "Venta por web",    
    "impuesto": "0.0",
    "total": "11200.0",
    "saldo": "9900.0",
    "cobro": 
    [{
	    "id": "455852",
	    "fecha_emision": "13/06/2023",
	    "tipo_cobro": "EF",
	    "total": "1200.0"
    }],
    "retencion":
    [{
      "id": "56051",
      "fecha_emision": "13/06/2023",
      "total": "100.0",
      "numero": "001-001-000005361",
      "autorizacion": "45678932598466966",
      "estado": "1"
    }]
}]
 
```
Los estados de documento son:

| Valor       | Tipo                                 |
| ----------- | ------------------------------------ |
| `1  `       | Activo                               |
| `0  `       | Anulado|

Atributos del objeto Documento:

| Parámetro   | Tipo    | Longitud | Obligatorio | Descripción |
| ----------- | ------- | -------- | ----------- | ----------- |
| `id` | varchar | 10 | Si | Identificador del cliente al que pertenece el documento en el sistema|
| `cliente_id` | varchar | 10 | Si | Identificador del cliente al que pertenece el documento en el sistema|
| `suscripcion_id` | varchar | 10 | Si | Identificador de la suscripción al que pertenece el documento en el sistema|
| `fecha_modificacion` | date | - | Si|Última fecha en la que se modificó|
| `fecha_emision ` | date | - | Si| Fecha de creación del documento|
| `fecha_vencimiento ` | date | - | Si| Fecha límite de vigencia del documento|
| `numero`|varchar|17|Si|Número del documento|
| `autorizacion`|varchar|49|Si|Número de autorización del documento|
| `estado`|varchar|10|Si|Representa el estado del documento (Activo o Anulado)|
| `descripcion`|varchar|100|Si|Representa la descripción del documento|
| `impuesto`|decimal|10|Si|Representa el valor del impuesto del documento|
| `total`|decimal|10|Si|Representa el valor total del documento|
| `saldo`|decimal|10|Si|Representa el valor del saldo del documento|
| `cobro `| objeto | - | No| Objeto contenedor del cobro del documento|
| `retencion`|objeto|-|No|Objeto contenedor de la retención del documento|

## Crear Documento (POST)

Para crear un documento se debe de hacer uso de la url:

`POST https://app.conpronto.com/documento/`

Por medio del método POST enviando en el cuerpo del requerimiento los datos del documento.

``` json title="Estructura del JSON:"
 {
    "cliente_id": "5662",
    "suscripcion_id": "74135",
    "fecha_emision": "13/06/2023",
    "fecha_vencimiento": "17/07/2023",
    "numero": "001-002-000000451",
    "autorizacion": "565645678932448598466966",
    "estado": "1",
    "descripcion": "Venta por web",      
    "impuesto": "0.0",
    "total": "11200.0",
    "saldo": "9900.0",
    "cobro":
    [{
	    "fecha_emision": "13/06/2023",
	    "tipo_cobro": "EF",
	    "total": "1200.0"
    }],
    "retencion":
    [{
      "fecha_emision": "13/06/2023",
      "total": "100.0",
      "numero": "001-001-000005361",
      "autorizacion": "45678932598466966",
      "estado": "1"
    }],
}
```
## Modificar Documento (PUT)

Este servicio permite modificar un documento creado por API.

Para modificar un documento se debe hacer uso de la url:

`PUT https://app.conpronto.com/documento/`

Los datos que se envían al momento de la actualización son los mismos que al momento de la creación aumentando el parámetro "id" dentro del json.

``` json title="Estructura del JSON:"
{
    "id": "7856",
    "cliente_id": "45248",
    "suscripcion_id": "74135",
    "fecha_emision": "13/06/2023",
    "fecha_vencimiento": "17/10/2023",
    "numero ": "001-001-000000251",
    "autorizacion": "45678932598466966",
    "estado": "1",
    "descripcion": "Venta por web",  
    "impuesto": "0.0",
    "total": "11200.0",
    "saldo": "9900.0",
    "cobro": 
    [{
	    "fecha_emision": "13/06/2023",
	    "tipo_cobro": "EF",
	    "total": "1200.0"
    }],
    "retencion": 
    [{
      "fecha_emision": "13/06/2023",
      "total": "100.0",
      "numero ": "001-001-000005361",
      "autorizacion": "45678932598466966",
      "estado": "1"
    }],
}
``` 

## Obtener un Documento (GET)

Para obtener un documento se debe de hacer uso de la url:

`GET https://app.conpronto.com/documento/<ID>`

Devuelve un documento con el <ID> solicitado.

Respuesta al consultar un documento:

``` json title="Respuesta al consultar un documento:"
[{
    "id": "7856",
    "cliente_id": "45248",
    "suscripcion_id": "74135",
    "fecha_emision": "13/06/2023",
    "fecha_vencimiento": "17/10/2023",
    "numero ": "001-001-000000251",
    "autorizacion": "45678932598466966",
    "estado": "1",
    "descripcion": "Venta por web",  
    "impuesto": "0.0",
    "total": "11200.0",
    "saldo": "9900.0",
    [{
	    "id": "7466",
	    "fecha_emision": "13/06/2023",
	    "tipo_cobro": "EF",
	    "total": "1200.0"
    }],
      "retencion": 
    [{
      "id": "7466",
      "fecha_emision": "13/06/2023",
      "total": "100.0",
      "numero": "001-001-000005361",
      "autorizacion": "45678932598466966",
      "estado": "1"
    }],
  }]
```

## Obtener Documentos por rango de fechas (GET)

Para obtener los documentos de un rango de fechas determinado hacemos uso del endpoint:

`GET https://app.conpronto.com/documento/?fecha_inicial=<FECHA_INICIAL>&fecha_final=<FECHA_FINAL>/`

Ejemplo:

`GET https://app.conpronto.com/documento/?fecha_inicial=1/02/2023&fecha_final=28/02/2023/`

``` json title="Respuesta al consultar todos los documentos:"
[{ 
    "id": "7466",
    "cliente_id": "52235",
    "suscripcion:id": "74135",
    "fecha_modificacion": "22/06/2023",
    "fecha_emision": "13/02/2023",
    "fecha_vencimiento": "17/10/2023",
    "numero ": "001-001-000000251",
    "autorizacion": "45678932598466966",
    "estado": "1",
    "descripcion": "Venta por web",  
    "impuesto": "0.0",
    "total": "11200.0",
    "saldo": "9900.0",
    "cobro": 
    [{
	    "id": "455852",
	    "fecha_emision": "13/06/2023",
	    "tipo_cobro": "EF",
	    "total": "1200.0"
    }],
    "retencion":
    [{
      "id": "56051",
      "fecha_emision": "13/06/2023",
      "total": "100.0",
      "numero": "001-001-000005361",
      "autorizacion": "45678932598466966",
      "estado": "1"
    }],
 },
 {
    "id": "5453",
    "cliente_id": "54522",
    "suscripcion_id": "325698",
    "fecha_modificacion": "22/06/2023",
    "fecha_emision": "25/02/2023",
    "fecha_vencimiento": "17/05/2023",
    "numero": "001-001-000000471",
    "autorizacion": "456789325984646466966",
    "estado": "1",
    "descripcion": "Venta al por mayor",
    "impuesto": "0.0",
    "total": "2000.0",
    "saldo": "1000.0",
    "cobro": 
    [{
	    "id": "455852",
	    "fecha_emision": "13/06/2023",
	    "tipo_cobro": "EF",
	    "total": "900.0"
    }],
    "retencion": 
    [{
      "id": "56051",
      "fecha_emision": "13/06/2023",
      "total": "100.0",
      "numero": "001-001-000005361",
      "autorizacion": "45678932598466966",
      "estado": "1"
    }],
 },
 ...
] 
```