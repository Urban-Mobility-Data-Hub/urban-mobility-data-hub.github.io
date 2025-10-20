# 🔐 Tokens y credenciales

El dataspace utiliza **OpenID for Verifiable Credentials (OID4VC)** y **OpenID for Verifiable Presentations (OID4VP)** para autenticar a los participantes mediante **credenciales verificables (VCs)**.  
A continuación se muestra cómo obtener una credencial verificable y un token OIDC para acceder a APIs protegidas.

---

## 1. Obtener una credencial verificable (OID4VC)

### Paso 1: Solicitar un Access Token inicial

```bash
export ACCESS_TOKEN=$(curl -s -k -x localhost:8888 -X POST \
  "$ISSUER_URL/realms/test-realm/protocol/openid-connect/token" \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data grant_type=password \
  --data client_id=account-console \
  --data username=operator \
  --data password=test | jq -r '.access_token')
```

### Paso 2: Obtener la oferta de credencial (Credential Offer URI)
```bash
export OFFER_URI=$(curl -s -k -x localhost:8888 -X GET \
  "$ISSUER_URL/realms/test-realm/protocol/oid4vc/credential-offer-uri?credential_configuration_id=operator_credential" \
  --header "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '"\(.issuer)\(.nonce)"')
```

### Paso 3: Obtener el código pre-autorizado
```bash
export PRE_AUTHORIZED_CODE=$(curl -s -k -x localhost:8888 -X GET ${OFFER_URI} \
  --header "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.grants."urn:ietf:params:oauth:grant-type:pre-authorized_code"."pre-authorized_code"')
```

### Paso 4: Intercambiar el código por un Credential Access Token
```bash
export CREDENTIAL_ACCESS_TOKEN=$(curl -s -k -x localhost:8888 -X POST \
  "$ISSUER_URL/realms/test-realm/protocol/openid-connect/token" \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code \
  --data pre-authorized_code=${PRE_AUTHORIZED_CODE} | jq -r '.access_token')
```

### Paso 5: Solicitar la credencial verificable (VC)
```bash
export VC_JWT=$(curl -s -k -x localhost:8888 -X POST \
  "$ISSUER_URL/realms/test-realm/protocol/oid4vc/credential" \
  --header "Authorization: Bearer ${CREDENTIAL_ACCESS_TOKEN}" \
  --header 'Content-Type: application/json' \
  --data '{"credential_identifier":"operator_credential", "format":"jwt_vc"}' \
  | jq -r '.credential')
```
## 2. Obtener un Access Token (OID4VP)
### Paso 1: Obtener el endpoint de token
```bash
export TOKEN_ENDPOINT=$(curl -s -k -X GET "$ISSUER_URL/.well-known/openid-configuration" \
  | jq -r '.token_endpoint')
```

### Paso 2: Crear una Verifiable Presentation (VP)

```bash
export HOLDER_DID=$(cat cert/did.json | jq -r '.id')
```
```bash
export VERIFIABLE_PRESENTATION=$(jq -nc \
  --arg vc "$VC_JWT" \
  --arg did "$HOLDER_DID" \
  '{ "@context": ["https://www.w3.org/2018/credentials/v1"],
     "type": ["VerifiablePresentation"],
     "verifiableCredential": [$vc],
     "holder": $did }')
```

### Paso 3: Construir y firmar el JWT (ES256)
```bash
header=$(jq -nc --arg kid "$HOLDER_DID" '{alg:"ES256", typ:"JWT", kid:$kid}')
payload=$(jq -nc --arg iss "$HOLDER_DID" --arg sub "$HOLDER_DID" \
  --argjson vp "$VERIFIABLE_PRESENTATION" '{iss:$iss, sub:$sub, vp:$vp}')

header_b64=$(echo -n "$header" | openssl base64 -A | tr '+/' '-_' | tr -d '=')
payload_b64=$(echo -n "$payload" | openssl base64 -A | tr '+/' '-_' | tr -d '=')
signing_input="${header_b64}.${payload_b64}"
signature_b64=$(echo -n "$signing_input" | openssl dgst -sha256 -binary -sign cert/private-key.pem | openssl base64 -A | tr '+/' '-_' | tr -d '=')

export JWT="${signing_input}.${signature_b64}"

```

### Paso 4: Solicitar el Access Token
```bash
export ACCESS_TOKEN=$(curl -s -k -x localhost:8888 -X POST ${TOKEN_ENDPOINT} \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data grant_type=vp_token \
  --data vp_token=${JWT} \
  --data scope=operator | jq -r '.access_token')
```

El resultado es una credencial verificable (VC) emitida por el Issuer y un Access Token OIDC derivado de ella, que se puede usar para acceder a endpoints protegidos por APISIX.