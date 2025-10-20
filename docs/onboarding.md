---
hide:
  - toc
  - navigation
---

## Conectar al Dataspace (Onboarding)

El proceso de conexión al dataspace define cómo un nuevo participante se incorpora al ecosistema de intercambio de datos de forma segura, confiable y verificable.

---

### Tipos de participantes

| Tipo | Descripción | Requisitos |
|------|--------------|-------------|
| **Provider** | Organización que ofrece activos de datos (*data assets*) a otros participantes del dataspace. | • Disponer de información corporativa verificada (nombre legal, dominio, CIF, etc.).<br>• Desplegar su propio **Provider connector**.<br>• Obtener una **Verifiable Credential (VC)** emitida por el *VC Issuer*.<br>• Registrar su endpoint y metadatos en el **Catálogo del Dataspace**. |
| **Consumer** | Entidad que desea consumir activos de datos ofrecidos por otros participantes. | • Obtener su **VC de identidad** emitida por el *VC Issuer*.<br>• No necesita desplegar infraestructura local: puede interactuar a través de la **Authorization API** o portales del nodo. |

---

## Flujo de incorporación

El proceso de onboarding es distinto según el tipo de participante. A continuación se describen ambos flujos de incorporación.



### Onboarding de **Consumer**

Antes de poder consumir datos dentro del dataspace, la organización consumidora debe estar **registrada en la Trusted Issuers List del proveedor**, de modo que se reconozca como un emisor de confianza de *Verifiable Credentials (VCs)* que incluyan los *claims* necesarios (por ejemplo, ser comprador autorizado de productos en el conector del proveedor).

![Flujo de onboarding del Consumer](assets/diagrama-onboarding.svg)

#### Flujo de registro del Consumer

1. **Solicitud de autenticación**  
   Un *Legal Entity Appointed Representative (LEAR)* de la organización consumidora accede al **Dataspace onboarding service** mediante una aplicación de contratación o portal web.  
   Este proceso comienza con la autenticación a través de su **wallet**, escaneando un código QR.

2. **Solicitud de credenciales verificables**  
   El **Verifier del provider** solicita al wallet del usuario las *Verifiable Credentials* necesarias que acrediten su identidad como **LEAR** y, eventualmente, otras VCs requeridas.

3. **Validación del Verifier**  
   El wallet comprueba que el Verifier pertenece a un participante legítimo del dataspace y devuelve las VCs solicitadas.

4. **Verificación de las credenciales**  
   El Verifier comprueba que la VC del LEAR fue emitida por un participante de confianza del dataspace, y valida que las demás VCs también procedan de **trusted issuers**.

5. **Emisión de token**  
   Si todas las verificaciones son correctas, el Verifier emite un **token OIDC**, que se devuelve al usuario.

6. **Registro de la organización**  
   Usando el token obtenido, el LEAR invoca las **TM Forum APIs** del proveedor para registrar la organización consumidora dentro del conector y establecer los permisos de acceso correspondientes.

7. **Adición a la Trusted Issuers List**  
   Una vez completado el registro, la organización queda incluida en la **Trusted Issuers List local** del proveedor como **emisor de confianza de VCs** con *claims* de tipo “USER”.





---

### Onboarding de **Provider**

El onboarding de un **Provider** tiene como objetivo registrar oficialmente a la organización como **participante de confianza** dentro del dataspace y habilitar su infraestructura para ofrecer datos.  
A diferencia del *Consumer*, el *Provider* debe desplegar su propio **Provider connector**.

![Flujo de onboarding del Provider](assets/diagrama-onboarding-provider.svg)

#### Flujo de registro del provider

#### 1. Verificación de identidad y conformidad

1. **Preparación de la autodescripción de la organización**  
   La organización genera su *Self-Description VC*, un documento estructurado que describe su identidad, capacidades y políticas de datos.

2. **Validación de conformidad con Gaia-X**  
   La *Self-Description VC* se envía al **Gaia-X Digital Clearing House (GXDCH)** para comprobar que cumple con las especificaciones de Gaia-X.  
   Si la validación es correcta, el GXDCH emite una **VC de conformidad**, que se almacena en el wallet del **LEAR** (Legal Entity Appointed Representative) de la organización.

3. **Solicitud de autenticación**  
   El LEAR accede al **portal de onboarding** o a la **Participant List App** del dataspace.  
   Al escanear el código QR de acceso con su wallet, el proceso se comunica con el **Verifier** del dataspace.

4. **Intercambio y validación de credenciales**  
   El Verifier solicita y valida las siguientes credenciales:
   - La **VC del LEAR**, que acredita al representante legal.  
   - La **Self-Description VC** de la organización.  
   - La **VC de conformidad** emitida por el GXDCH.  

   Si todas las VCs son válidas y emitidas por *trusted issuers*, el Verifier genera un **token OIDC** que autoriza el registro.

5. **Registro como participante de confianza**  
   Con ese token, la aplicación de onboarding invoca la **API del Participant List Service**, registrando la organización como **Trusted Participant** dentro del dataspace.

---

#### 2. Despliegue e integración del conector

Una vez registrada la organización, se habilita el entorno técnico que le permitirá ofrecer datos a otros participantes.

1. **Despliegue del conector**  
   La organización despliega su **EDC Provider Connector** (normalmente mediante Docker o Kubernetes) configurando:
   - **Endpoint público** (p. ej. `https://provider-a.dataspace.local`)  
   - **Integración con Keycloak** (realm, client ID y secret)  
   - **Integración con APISIX Gateway**  
   - **Configuración de políticas y permisos** en el **Policy Engine**

2. **Registro de activos y contratos**  
   Una vez operativo, el conector puede publicar **data assets** y **data offerings** en el catálogo del dataspace, estableciendo condiciones de acceso y contratos mediante las **TM Forum APIs**.




