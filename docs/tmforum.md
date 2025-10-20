# TM Forum APIs

El dataspace utiliza las APIs estándar de **TM Forum** para la gestión de ofertas, contratos y catálogos de productos.  
Estas APIs permiten registrar **especificaciones**, **ofertas** y **órdenes de producto**, además de **organizaciones** y **clientes**.

En esta sección se muestran los ejemplos más comunes.

---

## Crear una Product Specification

La **Product Specification** define las características técnicas y políticas que debe cumplir un producto o dataset publicado en el dataspace.  
Incluye información sobre credenciales requeridas y políticas ODRL asociadas.

```bash
export PRODUCT_SPEC_FULL_ID=$(curl -X POST \
  http://tm-forum-api.127.0.0.1.nip.io:8080/tmf-api/productCatalogManagement/v4/productSpecification \
  -H 'Content-Type: application/json;charset=utf-8' \
  -d "{
    \"brand\": \"M&P Operations\",
    \"version\": \"1.0.0\",
    \"lifecycleStatus\": \"ACTIVE\",
    \"name\": \"M&P K8S\",
    \"productSpecCharacteristic\": [
      {
        \"id\": \"credentialsConfig\",
        \"name\": \"Credentials Config\",
        \"@schemaLocation\": \"https://raw.githubusercontent.com/FIWARE/contract-management/refs/heads/main/schemas/credentials/credentialConfigCharacteristic.json\",
        \"valueType\": \"credentialsConfiguration\",
        \"productSpecCharacteristicValue\": [
          {
            \"isDefault\": true,
            \"value\": {
              \"credentialsType\": \"OperatorCredential\",
              \"claims\": [
                {
                  \"name\": \"roles\",
                  \"path\": \"$.roles[?(@.target==\\\"${PROVIDER_DID}\\\")].names[*]\",
                  \"allowedValues\": [\"OPERATOR\"]
                }
              ]
            }
          }
        ]
      },
      {
        \"id\": \"policyConfig\",
        \"name\": \"Policy for creation of K8S clusters.\",
        \"@schemaLocation\": \"https://raw.githubusercontent.com/FIWARE/contract-management/refs/heads/policy-support/schemas/odrl/policyCharacteristic.json\",
        \"valueType\": \"authorizationPolicy\",
        \"productSpecCharacteristicValue\": [
          {
            \"isDefault\": true,
            \"value\": {
              \"@context\": {\"odrl\": \"http://www.w3.org/ns/odrl/2/\"},
              \"@id\": \"https://mp-operation.org/policy/common/k8s-full\",
              \"odrl:uid\": \"https://mp-operation.org/policy/common/k8s-full\",
              \"@type\": \"odrl:Policy\",
              \"odrl:permission\": {
                \"odrl:assigner\": \"https://www.mp-operation.org/\",
                \"odrl:target\": {
                  \"@type\": \"odrl:AssetCollection\",
                  \"odrl:source\": \"urn:asset\",
                  \"odrl:refinement\": [{
                    \"@type\": \"odrl:Constraint\",
                    \"odrl:leftOperand\": \"ngsi-ld:entityType\",
                    \"odrl:operator\": \"odrl:eq\",
                    \"odrl:rightOperand\": \"K8SCluster\"
                  }]
                },
                \"odrl:assignee\": {
                  \"@type\": \"odrl:PartyCollection\",
                  \"odrl:source\": \"urn:user\",
                  \"odrl:refinement\": {
                    \"@type\": \"odrl:LogicalConstraint\",
                    \"odrl:and\": [
                      {
                        \"@type\": \"odrl:Constraint\",
                        \"odrl:leftOperand\": \"vc:role\",
                        \"odrl:operator\": \"odrl:hasPart\",
                        \"odrl:rightOperand\": {\"@value\": \"OPERATOR\", \"@type\": \"xsd:string\"}
                      },
                      {
                        \"@type\": \"odrl:Constraint\",
                        \"odrl:leftOperand\": \"vc:type\",
                        \"odrl:operator\": \"odrl:hasPart\",
                        \"odrl:rightOperand\": {\"@value\": \"OperatorCredential\", \"@type\": \"xsd:string\"}
                      }
                    ]
                  }
                },
                \"odrl:action\": \"odrl:use\"
              }
            }
          }
        ]
      }
    ]
  }" | jq -r '.id'); echo ${PRODUCT_SPEC_FULL_ID}
```

- Crea una Product Specification llamada M&P K8S

- Un bloque de credenciales que exige OperatorCredential con rol OPERATOR.

- Una política ODRL que limita su uso a entidades del tipo K8SCluster.

- Devuelve el identificador único (PRODUCT_SPEC_FULL_ID) que se usará al crear las ofertas.

---


### Listar especificaciones

```bash
curl -X GET http://tm-forum-api.127.0.0.1.nip.io:8080/tmf-api/productCatalogManagement/v4/productSpecification \
  -H 'Accept: application/json'
```
---

## Crear una Product Offering
Una Product Offering representa una oferta comercial concreta basada en una especificación previa.
Puede haber múltiples ofertas con diferentes condiciones, precios o políticas asociadas.

```bash
export PRODUCT_OFFERING_SMALL_ID=$(curl -X POST \
  http://tm-forum-api.127.0.0.1.nip.io:8080/tmf-api/productCatalogManagement/v4/productOffering \
  -H 'Content-Type: application/json;charset=utf-8' \
  -d "{
    \"version\": \"1.0.0\",
    \"lifecycleStatus\": \"ACTIVE\",
    \"name\": \"M&P K8S Offering Small\",
    \"productSpecification\": {
      \"id\": \"${PRODUCT_SPEC_FULL_ID}\"
    }
  }" | jq -r '.id'); echo ${PRODUCT_OFFERING_SMALL_ID}
```

- Crea una Product Offering basada en la Product Specification anterior.

- La asocia con el catálogo gestionado por el nodo provider.

- Devuelve el ID de la nueva oferta.


---
### Listar ofertas

```bash
curl -X GET http://tm-forum-api.127.0.0.1.nip.io:8080/tmf-api/productCatalogManagement/v4/productOffering \
  -H 'Accept: application/json'
```
---
## Registrar una organización (Organization)

Antes de poder realizar pedidos o firmar contratos, una organización consumidora debe registrarse en el conector del proveedor.  
Esto se realiza mediante la API **Party Management** de TM Forum (`/tmf-api/party/v4/organization`).

En este ejemplo, *Fancy Marketplace* se registra en *M&P Operations* utilizando una credencial de tipo **REPRESENTATIVE**.


```bash
export FANCY_MARKETPLACE_ID=$(curl -X POST \
  http://mp-tmf-api.127.0.0.1.nip.io:8080/tmf-api/party/v4/organization \
  -H 'Accept: */*' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -d "{
    \"name\": \"Fancy Marketplace Inc.\",
    \"partyCharacteristic\": [
      {
        \"name\": \"did\",
        \"value\": \"${CONSUMER_DID}\"
      }
    ]
  }" | jq -r '.id'); echo ${FANCY_MARKETPLACE_ID}
```

- Crea una Organization con nombre Fancy Marketplace Inc..

- Incluye el identificador descentralizado (DID) del consumidor en partyCharacteristic.

- La organización queda registrada dentro del catálogo del provider y podrá realizar pedidos o firmar acuerdos.

- Devuelve el identificador único de la organización (FANCY_MARKETPLACE_ID).

Una vez registrada la organización:

- El representative puede crear Product Orders usando su FANCY_MARKETPLACE_ID.

- El conector del proveedor puede aplicar las políticas ODRL que restringen qué roles (e.g. REPRESENTATIVE) pueden crear u ordenar productos.

---

## Consultar organizaciones registradas
```bash
curl -X GET \
  http://mp-tmf-api.127.0.0.1.nip.io:8080/tmf-api/party/v4/organization \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'Accept: application/json'
```
Devuelve una lista de todas las organizaciones registradas, incluyendo su id, name, y características (partyCharacteristic).

---

## Crear y completar un Product Order

Una vez registrada la organización y disponible una oferta (`Product Offering`), el consumidor puede **aceptar la oferta** creando un pedido (*Product Order*).  
Esto representa la firma de un acuerdo de uso de datos dentro del dataspace.

---

### Paso 1. Crear el pedido

El consumidor (por ejemplo, *Fancy Marketplace Inc.*) crea un **Product Order** indicando la oferta (`OFFER_ID`) que desea contratar y el identificador de su organización (`FANCY_MARKETPLACE_ID`).

```bash
export ORDER_ID=$(curl -X POST \
  http://mp-tmf-api.127.0.0.1.nip.io:8080/tmf-api/productOrderingManagement/v4/productOrder \
  -H 'Accept: */*' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -d "{
    \"productOrderItem\": [
      {
        \"id\": \"random-order-id\",
        \"action\": \"add\",
        \"productOffering\": { \"id\": \"${OFFER_ID}\" }
      }
    ],
    \"relatedParty\": [
      { \"id\": \"${FANCY_MARKETPLACE_ID}\" }
    ]
  }" | jq -r '.id'); echo ${ORDER_ID}
```

- Crea un Product Order asociado a la oferta seleccionada.

- El campo "action": "add" indica que se está solicitando la contratación del producto.

- relatedParty vincula el pedido con la organización consumidora.

- Devuelve el identificador único del pedido (ORDER_ID).

### Paso 2. Completar el pedido
Una vez procesado por el sistema del proveedor, el pedido debe marcarse como completado.
Esto simula la confirmación de que la organización tiene acceso al dataset o servicio adquirido.

```bash
curl -X PATCH \
  http://tm-forum-api.127.0.0.1.nip.io:8080/tmf-api/productOrderingManagement/v4/productOrder/${ORDER_ID} \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'Accept: application/json;charset=utf-8' \
  -H 'Content-Type: application/json;charset=utf-8' \
  -d '{
    "state": "completed"
  }' | jq .
```

- Actualiza el estado del pedido a "completed".

- Representa la finalización de la negociación o contrato.

- Una vez completado, se genera el Agreement que rige el acceso del consumidor a los activos de datos.

### Consultar pedidos activos
```bash
curl -X GET \
  http://mp-tmf-api.127.0.0.1.nip.io:8080/tmf-api/productOrderingManagement/v4/productOrder \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'Accept: application/json'
```
Todos los pedidos asociados al consumidor autenticado, junto con su estado (acknowledged, inProgress, completed, etc.).

Resultado final:

- El consumidor ha aceptado una oferta (Product Offering).

- El pedido ha sido procesado y completado.

- Se genera un Data Agreement que vincula las políticas ODRL y permite el acceso controlado a los datos en el dataspace.
