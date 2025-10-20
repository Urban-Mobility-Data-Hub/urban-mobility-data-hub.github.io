# Políticas y control de acceso

Las políticas determinan **quién puede acceder** a qué datos y bajo qué condiciones.  
Dentro del dataspace, las políticas se gestionan a través del **Policy Engine (OPA/ODRL-PAP)** y se aplican a los contratos y assets publicados.

---

## Crear una política ODRL

Ejemplo básico de política de acceso (en formato JSON):

```json
    {
        "@context": "https://www.w3.org/ns/odrl.jsonld",
        "uid": "urn:policy:read-access",
        "permission": [{
            "target": "urn:asset:temperature-sensor",
            "action": "use",
            "constraint": [{
            "leftOperand": "purpose",
            "operator": "eq",
            "rightOperand": "research"
            }]
        }]
    }
```

## Registrar la política
```json
    curl -X POST http://localhost:8080/odrl-pap/policies \
    -H "Content-Type: application/json" \
    -d @policy.json
```

##  Consultar políticas activas
```json
    curl http://localhost:8080/odrl-pap/policies
```
---

## Ejemplos de políticas
## Permitir a todos los participantes autenticados leer ofertas

Esta política permite que cualquier participante autenticado pueda consultar los ProductOfferings registrados en el catálogo.

```bash
curl -s -X POST http://pap-provider.127.0.0.1.nip.io:8080/policy \
  -H 'Content-Type: application/json' \
  -d '{
    "@context": {
      "odrl": "http://www.w3.org/ns/odrl/2/",
      "tmf": "https://tmforum.org/"
    },
    "odrl:uid": "https://mp-operation.org/policy/common/offering",
    "@type": "odrl:Policy",
    "odrl:permission": {
      "odrl:assigner": {"@id": "https://www.mp-operation.org/"},
      "odrl:target": {
        "@type": "odrl:AssetCollection",
        "odrl:source": "urn:asset",
        "odrl:refinement": [{
          "@type": "odrl:Constraint",
          "odrl:leftOperand": "tmf:resource",
          "odrl:operator": {"@id": "odrl:eq"},
          "odrl:rightOperand": "productOffering"
        }]
      },
      "odrl:assignee": {"@id": "vc:any"},
      "odrl:action": {"@id": "odrl:read"}
    }
  }'
```
## Permitir a los REPRESENTATIVE registrarse como cliente

Esta política restringe el registro de organizaciones únicamente a usuarios con el rol REPRESENTATIVE.

```bash
curl -s -X POST http://pap-provider.127.0.0.1.nip.io:8080/policy \
  -H 'Content-Type: application/json' \
  -d '{
    "@context": {"odrl": "http://www.w3.org/ns/odrl/2/", "tmf": "https://tmforum.org/"},
    "odrl:uid": "https://mp-operation.org/policy/common/selfRegistration",
    "@type": "odrl:Policy",
    "odrl:permission": {
      "odrl:assigner": {"@id": "https://www.mp-operation.org/"},
      "odrl:target": {
        "@type": "odrl:AssetCollection",
        "odrl:source": "urn:asset",
        "odrl:refinement": [{
          "@type": "odrl:Constraint",
          "odrl:leftOperand": "tmf:resource",
          "odrl:operator": {"@id": "odrl:eq"},
          "odrl:rightOperand": "organization"
        }]
      },
      "odrl:assignee": {
        "@type": "odrl:PartyCollection",
        "odrl:source": "urn:user",
        "odrl:refinement": {
          "@type": "odrl:LogicalConstraint",
          "odrl:and": [
            {
              "@type": "odrl:Constraint",
              "odrl:leftOperand": {"@id": "vc:role"},
              "odrl:operator": {"@id": "odrl:hasPart"},
              "odrl:rightOperand": {"@value": "REPRESENTATIVE", "@type": "xsd:string"}
            },
            {
              "@type": "odrl:Constraint",
              "odrl:leftOperand": {"@id": "vc:type"},
              "odrl:operator": {"@id": "odrl:hasPart"},
              "odrl:rightOperand": {"@value": "UserCredential", "@type": "xsd:string"}
            }
          ]
        }
      },
      "odrl:action": {"@id": "tmf:create"}
    }
  }'
```
## Permitir a los REPRESENTATIVE crear órdenes de productos

Esta política autoriza la creación de ProductOrders por parte de los usuarios con rol REPRESENTATIVE.

```bash
curl -s -X POST http://pap-provider.127.0.0.1.nip.io:8080/policy \
  -H 'Content-Type: application/json' \
  -d '{
    "@context": {"odrl": "http://www.w3.org/ns/odrl/2/", "tmf": "https://tmforum.org/"},
    "odrl:uid": "https://mp-operation.org/policy/common/ordering",
    "@type": "odrl:Policy",
    "odrl:permission": {
      "odrl:assigner": {"@id": "https://www.mp-operation.org/"},
      "odrl:target": {
        "@type": "odrl:AssetCollection",
        "odrl:source": "urn:asset",
        "odrl:refinement": [{
          "@type": "odrl:Constraint",
          "odrl:leftOperand": "tmf:resource",
          "odrl:operator": {"@id": "odrl:eq"},
          "odrl:rightOperand": "productOrder"
        }]
      },
      "odrl:assignee": {
        "@type": "odrl:PartyCollection",
        "odrl:source": "urn:user",
        "odrl:refinement": {
          "@type": "odrl:LogicalConstraint",
          "odrl:and": [
            {
              "@type": "odrl:Constraint",
              "odrl:leftOperand": {"@id": "vc:role"},
              "odrl:operator": {"@id": "odrl:hasPart"},
              "odrl:rightOperand": {"@value": "REPRESENTATIVE", "@type": "xsd:string"}
            },
            {
              "@type": "odrl:Constraint",
              "odrl:leftOperand": {"@id": "vc:type"},
              "odrl:operator": {"@id": "odrl:hasPart"},
              "odrl:rightOperand": {"@value": "UserCredential", "@type": "xsd:string"}
            }
          ]
        }
      },
      "odrl:action": {"@id": "tmf:create"}
    }
  }'
```
## Permitir a los OPERATORS leer información de clústeres

Esta política da acceso de lectura a entidades del tipo K8SCluster únicamente a usuarios con credenciales de tipo OperatorCredential.

```bash
curl -s -X POST http://pap-provider.127.0.0.1.nip.io:8080/policy \
  -H 'Content-Type: application/json' \
  -d '{
    "@context": {"odrl": "http://www.w3.org/ns/odrl/2/", "ngsi-ld": "https://uri.etsi.org/ngsi-ld/"},
    "odrl:uid": "https://mp-operation.org/policy/common/allowRead",
    "@type": "odrl:Policy",
    "odrl:permission": {
      "odrl:assigner": {"@id": "https://www.mp-operation.org/"},
      "odrl:target": {
        "@type": "odrl:AssetCollection",
        "odrl:source": "urn:asset",
        "odrl:refinement": [{
          "@type": "odrl:Constraint",
          "odrl:leftOperand": "ngsi-ld:entityType",
          "odrl:operator": {"@id": "odrl:eq"},
          "odrl:rightOperand": "K8SCluster"
        }]
      },
      "odrl:assignee": {
        "@type": "odrl:PartyCollection",
        "odrl:source": "urn:user",
        "odrl:refinement": {
          "@type": "odrl:LogicalConstraint",
          "odrl:and": [
            {
              "@type": "odrl:Constraint",
              "odrl:leftOperand": {"@id": "vc:role"},
              "odrl:operator": {"@id": "odrl:hasPart"},
              "odrl:rightOperand": {"@value": "OPERATOR", "@type": "xsd:string"}
            },
            {
              "@type": "odrl:Constraint",
              "odrl:leftOperand": {"@id": "vc:type"},
              "odrl:operator": {"@id": "odrl:hasPart"},
              "odrl:rightOperand": {"@value": "OperatorCredential", "@type": "xsd:string"}
            }
          ]
        }
      },
      "odrl:action": {"@id": "odrl:read"}
    }
  }'
```

Cada una de estas políticas se almacena en el Policy Administration Point (PAP) y es evaluada en tiempo real por el Policy Decision Point (PDP) cuando un usuario intenta realizar una operación sobre datos o recursos del dataspace.