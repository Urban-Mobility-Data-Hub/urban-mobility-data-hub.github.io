---
hide:
  - toc
  - navigation
---
El despliegue del Dataspace incluye un sistema de autenticación y autorización basado en **OIDC (OpenID Connect)** y **control de acceso por políticas (PDP/PAP/PIP/PEP)**.  
Este sistema garantiza que solo los participantes autorizados puedan acceder a los servicios y datos publicados dentro del entorno.

---

### Componentes principales

| Componente | Función |
|-------------|----------|
| **Keycloak** | Servicio de identidad (IdP) basado en **OIDC/OAuth2**. Gestiona usuarios, clientes y roles, emite tokens de acceso y actúa como punto central de autenticación. Se integra con el *VC Issuer* para firmar credenciales verificables (OID4VC). |
| **VC Issuer** | Servicio responsable de **emitir Verifiable Credentials (VCs)** conforme al estándar **OID4VC**. Firma digitalmente las credenciales emitidas por el dataspace y asocia la identidad del emisor con su autoridad (por ejemplo, una organización o nodo operador). |
| **VC Verifier** | Servicio encargado de **verificar la validez de las Verifiable Credentials** presentadas por los participantes. Comprueba la firma, el emisor y el estado de revocación para garantizar la confianza entre nodos. |
| **Policy Engine (PAP/PDP/PIP)** | Módulo de **autorización** basado en políticas (**XACML/OPA**). Evalúa si una solicitud cumple las condiciones de acceso definidas en contratos y políticas internas.<br><br>• **PAP:** administra las políticas.<br>• **PDP:** toma decisiones (*Permit/Deny*).<br>• **PIP:** obtiene información contextual (credenciales, contratos activos, roles). |


---

### Flujo de autenticación/autorización

1. El **usuario o sistema consumidor** accede a un servicio protegido (por ejemplo, una API expuesta por APISIX).  
2. **APISIX** redirige la solicitud hacia **Keycloak** para autenticación.  
3. Una vez autenticado, **Keycloak emite un token OIDC (JWT)** firmado.  
4. El token se adjunta a las peticiones posteriores (`Authorization: Bearer <token>`).  
5. **APISIX** valida la firma del token mediante VC Verifier y extrae los *claims* del usuario.  
6. Si las políticas de acceso lo permiten, la solicitud se enruta al servicio correspondiente.

---

### Roles básicos

| Rol | Descripción | Permisos principales |
|------|--------------|----------------------|
| **USER** | Participante estándar del dataspace. Puede consumir datos, negociar contratos y acceder a los recursos para los que tenga permisos explícitos. | • Consultar datasets públicos o contratados.<br>• Iniciar solicitudes de negociación.<br>• Acceder a APIs protegidas con token OIDC válido. |
| **REPRESENTATIVE** | Usuario con autoridad para actuar en nombre de una organización o entidad dentro del dataspace. Gestiona ofertas, contratos y credenciales verificables. | • Publicar *data offerings* y firmar contratos.<br>• Emitir o verificar *Verifiable Credentials* asociadas a la organización.<br>• Administrar políticas de acceso de sus activos. |
| **OPERATOR** | Administrador del nodo o del entorno de despliegue. Supervisa la infraestructura, la configuración de seguridad y los componentes del sistema. | • Desplegar y mantener los servicios (Keycloak, APISIX, EDC, etc.).<br>• Crear y gestionar realms, clientes y usuarios.<br>• Monitorizar logs, métricas y políticas globales. |


