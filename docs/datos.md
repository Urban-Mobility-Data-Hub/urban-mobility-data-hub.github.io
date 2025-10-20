## Acceso protegido mediante APISIX

Los endpoints de datos del dataspace están protegidos por el **API Gateway (APISIX)**, que actúa como **Policy Enforcement Point (PEP)**.  
Esto significa que cualquier operación sobre datos (lectura o escritura) requiere un **token OIDC válido** emitido por el sistema de autenticación.

---

## Operaciones disponibles en `/ngsi-ld/v1/entities`

El **Context Broker (Scorpio)** implementa el estándar **NGSI-LD**, lo que permite gestionar entidades semánticas con operaciones CRUD (Create, Read, Update, Delete).  
Todas estas llamadas pueden hacerse a través del gateway **APISIX**, autenticadas mediante **Bearer Token OIDC**.

---

### Obtener un token de acceso (OID4VP)

Antes de interactuar con los datos, es necesario obtener un **Access Token** autenticado mediante el flujo **OID4VP** (*OpenID for Verifiable Presentations*).

Ejemplo de obtención de token:

```bash
    export ACCESS_TOKEN=$(./doc/scripts/get_access_token_oid4vp.sh \
    http://mp-data-service.127.0.0.1.nip.io:8080 \
    $OPERATOR_CREDENTIAL \
    operator)
```
---

### Crear una entidad (Create)

Una vez obtenido el token, puedes crear una entidad NGSI-LD dentro del Scorpio Broker, a través del endpoint protegido de APISIX:

```bash
    curl -X POST http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/entities \
    -H 'Accept: */*' \
    -H 'Content-Type: application/json' \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -d '{
        "id": "urn:ngsi-ld:K8SCluster:fancy-marketplace",
        "type": "K8SCluster",
        "name": {
        "type": "Property",
        "value": "Fancy Marketplace Cluster"
        },
        "numNodes": {
        "type": "Property",
        "value": "3"
        },
        "k8sVersion": {
        "type": "Property",
        "value": "1.26.0"
        }
    }'
```
---

### Consultar todas las entidades (Read All)

```bash
    curl -X GET http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/entities \
    -H "Accept: application/json" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

Devuelve una lista de todas las entidades visibles para el usuario autenticado (según las políticas aplicadas).

### Consultar una entidad específica (Read by ID)

```bash
    curl -X GET http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/entities/urn:ngsi-ld:K8SCluster:fancy-marketplace \
    -H "Accept: application/json" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

Obtiene los detalles de una entidad concreta, incluyendo sus propiedades y relaciones.

### Consultar por tipo o atributos (Query)
```bash
    curl -G http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/entities \
    -H "Accept: application/json" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    --data-urlencode 'type=K8SCluster'
```

Puedes usar parámetros como:

- type — filtra por tipo de entidad.

- q — aplica condiciones sobre atributos (q=numNodes>2).

- attrs — selecciona atributos específicos.

- options=keyValues — devuelve un formato simplificado.


### Actualizar atributos (Update)
```bash
    curl -X PATCH http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/entities/urn:ngsi-ld:K8SCluster:fancy-marketplace/attrs \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -d '{
        "numNodes": { "type": "Property", "value": 5 }
    }'
```

Modifica uno o varios atributos de una entidad existente.

### Eliminar una entidad (Delete)
```bash
    curl -X DELETE http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/entities/urn:ngsi-ld:K8SCluster:fancy-marketplace \
    -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

Elimina la entidad indicada del Context Broker (si la política de acceso lo permite).

## Suscripciones (NGSI-LD Subscriptions)

Las suscripciones permiten recibir notificaciones automáticas cuando una entidad del **Context Broker (Scorpio)** cambia, cumple ciertas condiciones o deja de existir.  
Cada suscripción define:

- Qué entidades o tipos quiere observar.  
- Qué condiciones deben cumplirse.  
- A qué endpoint se deben enviar las notificaciones.

---

### Crear una suscripción

```bash
curl -X POST http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/subscriptions \
  -H "Content-Type: application/ld+json" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -d '{
    "id": "urn:ngsi-ld:Subscription:K8SClusterUpdates",
    "type": "Subscription",
    "entities": [{
      "type": "K8SCluster"
    }],
    "watchedAttributes": ["numNodes"],
    "q": "numNodes>2",
    "notification": {
      "endpoint": {
        "uri": "http://consumer-service.127.0.0.1.nip.io:8080/notify",
        "accept": "application/json"
      },
      "format": "normalized"
    },
    "throttling": 5
  }'
```

Qué hace este ejemplo:

- Observa todas las entidades de tipo K8SCluster.

- Solo envía notificaciones si el atributo numNodes cambia y cumple numNodes>2.

- Envia un POST al endpoint http://consumer-service.../notify.

- Usa throttling: 5 para no enviar más de una notificación cada 5 s.

### Consultar suscripciones existentes
```bash
curl -X GET http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/subscriptions \
  -H "Accept: application/json" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```
Devuelve todas las suscripciones visibles para el usuario autenticado.

### Consultar una suscripción específica
```bash
curl -X GET http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/subscriptions/urn:ngsi-ld:Subscription:K8SClusterUpdates \
  -H "Accept: application/json" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```
### Eliminar una suscripción
```bash
curl -X DELETE http://mp-data-service.127.0.0.1.nip.io:8080/ngsi-ld/v1/subscriptions/urn:ngsi-ld:Subscription:K8SClusterUpdates \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

### Notificaciones al consumidor
Cuando se cumpla la condición definida, Scorpio enviará una notificación al endpoint configurado:

```json
{
  "id": "urn:ngsi-ld:K8SCluster:fancy-marketplace",
  "type": "K8SCluster",
  "numNodes": { "type": "Property", "value": 4 },
  "observedAt": "2025-10-14T09:30:00Z",
  "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"
}
```
El servicio receptor (/notify) debe responder con 200 OK para confirmar la recepción.

### Notas

Todas las operaciones pasan por APISIX, que valida el token OIDC con Keycloak/Verifier.

El Policy Engine (OPA) determina si el usuario tiene permiso para leer o modificar las entidades solicitadas.

Por defecto todo es 403 Unathorized, si no se crea una política establecida que permita hacer algo.