---
hide:
  - toc
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
