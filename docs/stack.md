---
hide:
  - toc
  - navigation
---

![Diagrama de componentes](assets/diagrama-componentes.svg)

Urban Mobility Data Hub (UMDH) se construye sobre una arquitectura modular que implementa los principios de los Data Spaces, garantizando interoperabilidad, trazabilidad y control de acceso en el intercambio de datos.

## Componentes principales

| Componente | Rol | Campo en el diagrama | Enlace |
|-------------|-----|----------------------|---------|
| **VCVerifier** | Valida las *Verifiable Credentials (VCs)* y las intercambia por tokens de acceso. | Verifier | [FIWARE/VCVerifier](https://github.com/FIWARE/VCVerifier) |
| **credentials-config-service** | Contiene la información sobre qué VCs son necesarias para acceder a cada servicio. | PRP/PAP (autenticación) | [FIWARE/credentials-config-service](https://github.com/FIWARE/credentials-config-service) |
| **Keycloak** | Emisor de *Verifiable Credentials* en el lado del consumidor y proveedor de identidad (IdP) basado en OIDC/OAuth2. | Issuer | [keycloak.org](https://www.keycloak.org) |
| **Scorpio (Orion-LD)** | *Context Broker* encargado de la gestión semántica de datos mediante NGSI-LD. | Context Broker | [ScorpioBroker](https://github.com/ScorpioBroker/ScorpioBroker) |
| **trusted-issuers-list** | Actúa como *Trusted Issuers List*, ofreciendo una API compatible con el registro de emisores de confianza (EBSI Trusted Issuers Registry API). | Lista local de emisores de confianza | [FIWARE/trusted-issuers-list](https://github.com/FIWARE/trusted-issuers-list) |
| **APISIX** | Pasarela API (API Gateway) que funciona como **PEP (Policy Enforcement Point)** e integra un plugin de OPA para control de acceso. | PEP | [apisix.apache.org](https://apisix.apache.org/) / [plugin OPA](https://apisix.apache.org/docs/apisix/plugins/opa/) |
| **OPA (Open Policy Agent)** | Motor de políticas que actúa como **PDP (Policy Decision Point)** en el *sidecar* del API Gateway. Evalúa las políticas de autorización. | PDP | [openpolicyagent.org](https://www.openpolicyagent.org/) |
| **odrl-pap** | Permite configurar políticas ODRL que serán utilizadas por OPA para la autorización. | PRP/PAP (autorización) | [wistefan/odrl-pap](https://github.com/wistefan/odrl-pap) |
| **tmforum-api** | Implementación de las APIs de TM Forum para la gestión de contratos, ofertas y productos. | Gestión de contratos | [FIWARE/tmforum-api](https://github.com/FIWARE/tmforum-api) |
| **contract-management** | Servicio de escucha y notificación de eventos relacionados con la gestión de contratos provenientes de TM Forum. | Gestión de contratos | [FIWARE/contract-management](https://github.com/FIWARE/contract-management) |
| **MySQL** | Base de datos relacional utilizada para almacenamiento estructurado de configuraciones o contratos. | Base de datos | [mysql.com](https://www.mysql.com) |
| **PostgreSQL** | Base de datos relacional avanzada para almacenamiento estructurado. | Base de datos | [postgresql.org](https://www.postgresql.org) |
| **PostGIS** | Extensión espacial de PostgreSQL que añade soporte geoespacial y consultas sobre datos geográficos. | Base de datos | [postgis.net](https://postgis.net/) |


---



![Diagrama de apis](assets/diagrama-general-apis.svg)

---

## APIs principales

| API | Rol | Responsabilidad | Ejemplos de Endpoints |
|------|-----|-----------------|------------------------|
| **Data API** | Gestión de *data assets* (recursos reales de datos). | Registrar, exponer y servir datos compartidos dentro del Dataspace.<br><br>Incluye:<br>• Entidades NGSI-LD, datasets o flujos de datos.<br>• Acceso a los endpoints reales.<br>• CRUD de assets (crear, consultar, actualizar, eliminar).<br>• Servicios de descubrimiento por ID o tipo de recurso. | `/data/assets` → lista de datasets disponibles.<br>`/data/entities` → acceso a datos NGSI-LD. |
| **Contract API** | Gestión de ofertas y contratos de uso de los datos. | Orquestar el ciclo de vida de los *data offerings* desde su definición hasta la negociación.<br><br>Incluye:<br>• ProductSpecification, ProductOffering y Agreement (TM Forum).<br>• Registro de ofertas disponibles y condiciones.<br>• Iniciar, aceptar o rechazar negociaciones.<br>• Generar Data Agreements y vincularlos a Data Assets. | `/contract/offerings` → lista de ofertas.<br>`/contract/agreements` → contratos activos.<br>`/contract/negotiations` → gestión de negociaciones. |
| **Authorization API** | Control de acceso y aplicación de políticas (OPA/PDP). | Evaluar si una solicitud cumple las condiciones de seguridad y contrato.<br><br>Incluye:<br>• Integración con OPA (Open Policy Agent).<br>• Evaluación de políticas (PDP/PIP/PAP).<br>• Validación de tokens OIDC y claims.<br>• Envío de decisiones “permit/deny” a APISIX. | `/authz/evaluate` → recibe token y contexto, devuelve *permit/deny*. |
| **Authentication API** | Identidad y emisión de tokens o credenciales. | Gestionar la autenticación de usuarios, aplicaciones o conectores.<br><br>Incluye:<br>• Integración con Keycloak (OIDC, OAuth2).<br>• Emisión y validación de tokens JWT.<br>• Verifiable Credentials (OID4VC).<br>• Endpoints de login, refresh, revoke. | `/authn/token` → login.<br>`/authn/validate` → validar token. |
